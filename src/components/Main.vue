<template>
    <div>
        <h1 style="text-align: center; color:blue"><i class="bi bi-bag-heart-fill"></i>Secret Code</h1>
    </div>
    <div class="tool" :class="{'tool-sticky': toolSticky}">
        <i v-for="n in optionNumber" :class="options[n-1]" @click="insert(n-1)"></i>
        <button type="button" class="btn" @click="toolSticky = !toolSticky">
            <i class="bi" :class="{'bi-geo-fill': toolSticky, 'bi-geo': !toolSticky}"></i>
        </button>
    </div>
    <div>
        <table class="table">
            <thead>
                <tr>
                    <th></th>
                    <th v-for="n in codeNumber" style="text-align: center;">
                        <i v-if="state == 0" class="bi bi-circle icon"></i>
                        <i v-if="state == 1" class="bi bi-question-circle icon"></i>
                        <i v-if="state == 2" :class="options[secretCode[n - 1]]"></i>
                    </th>
                    <th style="text-align: center;" width="200px">
                        <button v-if="state == 1" class="btn btn-danger" @click="end">
                            <i class="bi bi-stop-circle-fill icon"></i>
                        </button>
                        <button v-else class="btn btn-primary" @click="start">
                            <i class="bi bi-play-circle-fill icon"></i>
                        </button>
                        <!-- <span>{{ usedTime }}</span> -->
                    </th>
                </tr>
            </thead>
            <tbody>
                <tr v-for="(row, rowIndex) in rows" :key="rowIndex">
                    <td class="text-center">{{ rowIndex + 1 }}</td>
                    <td v-for="(value, colIndex) in row" :class="{'table-primary': (rowIndex == activeRow && colIndex == activeColumn)}" @click="click(rowIndex, colIndex)">
                        <i v-if="value < 0 && rowIndex == activeRow" class="bi bi-circle icon icon-ready"></i>
                        <i v-else-if="value < 0" class="bi bi-circle icon icon-blank"></i>
                        <i v-else :class="options[value]"></i>
                    </td>
                    <td>
                        <button v-if="rowState[rowIndex] == 1" class="btn btn-primary" @click="submit(rowIndex)">提交</button>
                        <span v-if="rowState[rowIndex] == 2 && rowResult[rowIndex][0] < codeNumber">
                            <i v-for="n in rowResult[rowIndex][0]" class="bi bi-emoji-laughing icon-result1"></i>
                            <i v-for="n in rowResult[rowIndex][1]" class="bi bi-emoji-smile icon-result2"></i>
                        </span>
                        <span v-if="rowState[rowIndex] == 2 && rowResult[rowIndex][0] == codeNumber">
                            <i class="bi bi-emoji-heart-eyes-fill icon-result-success"></i>
                            <i class="bi bi-emoji-kiss-fill icon-result-success"></i>
                            <i class="bi bi-emoji-sunglasses-fill icon-result-success"></i>
                        </span>
                    </td>
                </tr>
            </tbody>
        </table>
        
    </div>
</template>

<script>
const N = 5;    // 多少个密码
const M = 12;   // 最多可以猜多少次
// 总共多少个选项，选项的内容用颜色表示
const OPTIONS = [
    'bi icon bi-circle-fill icon1',
    'bi icon bi-circle-fill icon2',
    'bi icon bi-circle-fill icon3',
    'bi icon bi-circle-fill icon4',
    'bi icon bi-circle-fill icon5',
    'bi icon bi-circle-fill icon6',
    'bi icon bi-circle-fill icon7',
    'bi icon bi-circle-fill icon8',
];

// 随机生成一个密码
function randomSecretCode() {
    var result = [];
    var options = [];
    for (var i = 0; i < OPTIONS.length; i++) {
        options[i] = i;
    }
    for (var i = 0; i < N; i++) {
        const randomIndex = Math.floor(Math.random() * options.length);
        const randomElement = options.splice(randomIndex, 1)[0];
        result[i] = randomElement;
    }

    console.log(result);
    return result;
}

export default {
    setup() {
    },

    // Properties returned from data() become reactive state
    // and will be exposed on `this`.
    data() {
        // 用户猜测信息初始化
        var rows = [];
        for (var i = 0; i < M; i++) {
            rows[i] = [];
            for (var j = 0; j < N; j++) {
                rows[i][j] = -1;
            }
        }

        var rowResult = [];
        var rowState = [];
        for (var i = 0; i < M; i++) {
            rowState[i] = 0;
            rowResult[i] = [];
        }

        var secretCode = [];
        for (var i = 0; i < N; i++) {
            secretCode[i] = -1;
        }

        return {
            codeNumber: N,
            options: OPTIONS,
            optionNumber: OPTIONS.length,
            secretCode: secretCode, // 密码
            state: 0,               // 全局游戏状态，0初始化，1进行中，2结束
            rows: rows,             // 用户猜测信息，N列，M行
            rowState: rowState,     // 行状态，0未标记完整，1标记完整，2已提交
            rowResult: rowResult,   // 行结果，[全对个数，颜色对个数]
            startTime: new Date(),           // 开始时间
            endTime: 0,             // 结束时间
            usedTime: "00:00:00",   // 用了多少时间
            activeRow: -1,          // 活动行
            activeColumn: -1,       // 活动列
            toolSticky: false,       // 工具条固定
        };
    },

    // Methods are functions that mutate state and trigger updates.
    // They can be bound as event handlers in templates.
    methods: {
        // 开始游戏
        start() {
            this.state = 1;
            this.rowState = [];
            this.rowResult = [];
            this.secretCode = randomSecretCode();

            // 清空上次游戏结果
            for (var i = 0; i < this.rows.length; i++) {
                for (var j = 0; j < this.rows[i].length; j++) {
                    this.rows[i][j] = -1;
                }
            }

            this.startTime = new Date();
            this.endTime = 0;
            this.activeRow = 0;
            this.activeColumn = 0;
        },
    
        // 结束游戏
        end() {
            if (!confirm("确定要结束当前游戏吗！")) {
                return;
            }
            this._end();
        },

        _end() {
            this.state = 2;
            this.endTime = new Date();
            this.activeRow = -1;
            this.activeColumn = -1;
        },

        // 点击表格单元格
        click(i, j) {
            if (this.activeRow == i) { // 活跃行
                if (this.activeColumn == j) {
                    this.rows[i][j] = -1;
                }
                else {
                    this.activeColumn = j;
                }
            }
            else if (this.rows[i][j] >= 0) { // 非活跃行，有颜色，拷贝插入到活跃位置
                this.insert(this.rows[i][j]);
            }
            
            this.validateRow();
        },

        // 向表格中插入元素
        insert(value) {
            if (this.activeRow < 0 || this.activeColumn < 0) {
                return;
                
            }

            this.rows[this.activeRow][this.activeColumn] = value;

            // 并且把同行中其他列的同样颜色去除
            for (var j = 0; j < this.rows[this.activeRow].length; j++) {
                if (j != this.activeColumn && this.rows[this.activeRow][j] == value) {
                    this.rows[this.activeRow][j] = -1;
                }
            }

            this.validateRow();
        },

        validateRow() {
            // 标记是否完整
            var rowState = 1;
            for (var j = 0; j < this.rows[this.activeRow].length; j++) {
                if (this.rows[this.activeRow][j] < 0) {
                    rowState = 0;
                    break;
                }
            }

            this.rowState[this.activeRow] = rowState;
        },

        submit(i) {
            if (this.rowState[i] != 1) {
                return;
            }

            // 标记不能重复
            if (new Set(this.rows[i]).size != this.rows[i].length) {
                alert("颜色不能重复，请调整！")
                return;
            }

            // if (!confirm("确定提交第" + (i+1) + "行！")) {
            //     return;
            // }

            var result1 = 0, result2 = 0;
            for (var j = 0; j < this.rows[i].length; j++) {
                if (this.rows[i][j] == this.secretCode[j]) {
                    result1++;
                }
                else {
                    var matchColor = false;
                    for (var k = 0; k < this.secretCode.length; k++) {
                        if (this.rows[i][j] == this.secretCode[k]) {
                            matchColor = true;
                            break;
                        }
                    }
                    if (matchColor) {
                        result2++;
                    }
                }
            }

            this.rowResult[i] = [result1, result2];
            this.rowState[i] = 2;

            // 活跃到下一行
            this.activeRow++;
            this.activeColumn = 0;

            // 游戏通关了，结束
            if (result1 == N) {
                this._end();
            }
        }
    },

    // Lifecycle hooks are called at different stages
    // of a component's lifecycle.
    // This function will be called when the component is mounted.
    mounted() {
        setInterval(() => {
            var now = new Date();
            
            if (this.state == 0) {
                this.usedTime = '00:00:00';
            }
            else if (this.state == 1) {
                this.usedTime = formatTime(Math.floor((now - this.startTime) / 1000));
            }
            else {
                this.usedTime = formatTime(Math.floor((this.endTime - this.startTime) / 1000));
            }
        }, 1000); // 每秒更新一次
    }
};

// 格式化时间
function formatTime(seconds) {
    const hours = Math.floor(seconds / 3600);
    const minutes = Math.floor((seconds % 3600) / 60);
    const remainingSeconds = seconds % 60;

    const formattedHours = String(hours).padStart(2, '0');
    const formattedMinutes = String(minutes).padStart(2, '0');
    const formattedSeconds = String(remainingSeconds).padStart(2, '0');

    return `${formattedHours}:${formattedMinutes}:${formattedSeconds}`;
}
</script>
  