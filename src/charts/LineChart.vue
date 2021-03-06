<script>
/**
 * @file UserActivityPlot.vue
 *
 * @brief The components that plot user activity with line gradient
 *
 * The props data should look like:
 *     [["2001-01", 12], ["2001-02", 23], ..., ["2001-12", 23]]
 * Reference:
 *      https://www.jianshu.com/p/7994176fbcc4
 *      http://www.echartsjs.com/examples/editor.html?c=line-gradient
 *
 * @author He, Hao; Shigma
 * @since 2019-01-07
 */

const eventBus = require('../EventBus')
const { debounce } = require('throttle-debounce')

const staticOptions = {
    visualMap: {
        show: false,
        type: 'continuous',
        seriesIndex: 0,
        min: 0,
    },
    grid: {
        top: 25,
        bottom: 20,
    },
    tooltip: {
        trigger: 'axis',
    },
    toolbox: {
        feature: {
            brush: {
                type: ['lineX', 'clear'],
            },
        },
    },
    brush: {
        xAxisIndex: 'all',
        throttleType: 'debounce',
        throttleDelay: 50000,
        outOfBrush: {
            colorAlpha: 0.1,
        },
    },
    yAxis: {
        splitLine: { show: false },
    },
    series: {
        name: '邮件数',
        type: 'line',
        showSymbol: false,
        itemStyle: {
            normal: {
                color: '#d68262'
            }
        },
        areaStyle: {
            normal: {
                color: new echarts.graphic.LinearGradient(0, 0, 0, 1, [{
                    offset: 0,
                    color: '#d68262'
                }, {
                    offset: 1,
                    color: '#ffe'
                }])
            }
        },
    },
}

module.exports = {
    extends: require('.'),
    props: ['startDate', 'endDate', 'hideTitle'],
    computed: {
        maxValue() {
            return this.data.reduce((total, curr) => {
                return total > curr[1] ? total : curr[1];
            });
        },
    },
    mounted() {
        this.chart.on('brush', debounce(100, params => {
            // null if no range is chosen
            this.startDate = null;
            this.endDate = null;

            if (params.areas[0]) {
                let range = params.areas[0].coordRange;
                if (range[0] > 0 && range[0] < this.data.length) {
                    this.startDate = this.data[range[0]][0];
                }
                if (range[1] > 0 && range[1] < this.data.length) {
                    this.endDate = this.data[range[1]][0];
                }
            }

            this.$emit('update:startDate', this.startDate)
            this.$emit('update:endDate', this.endDate)
        }))
    },
    methods: {
        setOption() {
            if (!this.chart) return
            const options = {
                ...staticOptions,
                visualMap: {
                    ...staticOptions.visualMap,
                    max: this.maxValue,
                },
                xAxis: {
                    data: this.data.map(item => item[0]),
                },
                series: {
                    ...staticOptions.series,
                    data: this.data.map(item => item[1]),
                },
            }
            if (this.hideTitle === undefined) {
                options.title = {
                    left: 'center',
                    text: 'Activity',
                    textStyle: {
                        fontSize: 18,
                    }
                };
            }
            this.chart.setOption(options);
        },
    },
}

</script>

<template>
    <div class="line-chart"/>
</template>

<style lang="scss" scoped>
</style>
