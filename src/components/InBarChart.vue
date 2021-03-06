<template>
    <div class="in-bar-wrapper">
        <svg
            class="root"
            ref="svg"
            :width="width"
            :height="height">

            <svg class="croped" :width="width" :height="height">
                <defs>
                    <linearGradient v-for="(g, i) in gradients" :key="`gradient-${i}`" :id="`gradient-${i}`" :x1="g.x1" :y1="g.y1" :x2="g.x2" :y2="g.y2">
                        <stop v-for="(s, j) in g.stops" :key="`stop-${i}-${j}`" :offset="s.offset" :style="`stop-color:${s.color}`" />
                    </linearGradient>
                </defs>

                <g class="bars-group">
                    <rect class="bar" 
                        v-for="(b, i) in barsComp" :key="`bar-${i}`"
                        :x="b.x"
                        :y="b.y"
                        :fill="b.fillColor"
                        :height="b.height"
                        :width="b.width"
                        :stroke="strokeColor"
                        :stroke-width="strokeWidth"
                        @mousemove="drawTooltip(b)"
                        @mouseout="isShowingTooltip = false"
                    />
                </g>

                <polyline class="line-zero" :points="`0,${zeroHeight} ${width},${zeroHeight}`" stroke="#aaa" stroke-width="1" fill="none" stroke-dasharray="1, 2" />
            </svg>

            <svg :x="tooltip.x" :y="tooltip.y" class="tooltip" ref="tooltip" :style="`overflow: initial; visibility: ${isShowingTooltip ? 'visible' : 'hidden'}`">
                <g>
                    <rect :width="tooltip.width || 100" :height="20" fill="rgba(255, 255, 255, 0.7)" stroke="#ccc" />
                    <text x="3" y="10" font-size="14" font-family="monospace" ref="tooltipText" alignment-baseline="central"> {{ tooltip.text }} </text>
                </g>
            </svg>

        </svg>
    </div>
</template>

<script>

    import ChartUtils from '../utils/ChartUtils';

    export default {
        props: {
            width:       { type: String,  default:  '250'   },
            height:      { type: String,  default:  '100'   },
            bars:        { type: Array,   required: true    },
            fillColor:   {                default: 'tomato' },
            strokeWidth: { type: Number,  default:  1       },
            strokeColor: { type: String,  default:  '#333'  },
            padding:     { type: Number,  default:  0       },
            minY:        { type: Number,  default:  null    },
            maxY:        { type: Number,  default:  null    },
            showTooltip: { type: Boolean, default:  true    }
        },
        data() {
            return {
                zeroHeight: 0,
                gradients: [],
                isShowingTooltip: false,
                tooltip: {
                    x: 0,
                    y: 0,
                    text: ''
                }
            }
        },
        methods: {
            drawTooltip(bar) {
                if (!this.isShowingTooltip && this.showTooltip) {
                    this.isShowingTooltip = true;
                    let text = bar.value;
                    let tooltipWidth = text.toString().length * 8 + 6;
                    let barWidth = bar.width;

                    let barCenter = bar.x + barWidth / 2;
                    let tooltipX = barCenter - tooltipWidth / 2;

                    this.tooltip = {
                        x: tooltipX,
                        y: bar.y - 22,
                        text: text,
                        width: tooltipWidth
                    };
                }
            }
        },
        computed: {
            fillColorComp() {
                let fillColor = this.fillColor;
                return ChartUtils.getSvgColorProperty(fillColor, this.gradients);
            },
            barsComp() {
                let self = this;

                let maxValue = null;
                let minValue = null;
                const strokePadding = this.strokeWidth;

                // Color property
                const fillColor = this.fillColorComp;

                // Finding the min and max Y values to calculate the yFactor and zero line position
                minValue  =              this.bars.reduce((min, b) => (b.value || b) < min || min == null ? (b.value || b) : min, minValue);
                maxValue  = this.maxY || this.bars.reduce((max, b) => (b.value || b) > max || max == null ? (b.value || b) : max, maxValue);

                const h = parseInt(this.height) - 2 * strokePadding;
                const w = parseInt(this.width) - 2 * strokePadding;

                let yTranslate = typeof this.minY === 'undefined' || this.minY === null ? Math.min(0, minValue) : this.minY;

                // This is how much 1 unit represents in pixels
                let yFactor = h / (maxValue - yTranslate);

                // Calculating the height of the zero line
                this.zeroHeight = h - ((0 - yTranslate) * yFactor) + strokePadding;

                // Calculate the width of each bar
                let barWidth = w / this.bars.length;

                // Calculating the bars
                let bars = [];
                let xStep = barWidth;
                let xCurr = strokePadding;
                this.bars.forEach(b => {
                    let value = b.value || b;
                    let barColor = undefined;

                    if (b.fillColor)
                        barColor = ChartUtils.getSvgColorProperty(b.fillColor, self.gradients);

                    bars.push({
                        value: value,
                        width: barWidth - 2 * self.padding,
                        height: Math.abs(value * yFactor),
                        x: xCurr + self.padding,
                        y: value > 0 ? h - ((value - yTranslate) * yFactor) + strokePadding : self.zeroHeight,
                        fillColor: barColor || fillColor
                    });
                    xCurr += xStep;
                });

                return bars;
            }
        },
        beforeMount() {
            
        }
    }
</script>

<style lang="scss" scoped>

    .in-bar-wrapper {
        .root {
            overflow: initial;
            .croped {

                .bar {
                    cursor: pointer;
                    transition: all .3s;

                }
            }

        }
    }

</style>
