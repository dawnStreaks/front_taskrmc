<script>
    import {Doughnut} from 'vue-chartjs'
    import {store} from '@/components/gaugecharts/store';

    export default {
        extends: Doughnut,
        store: store,
        props: ['chartdata','improved'],
          beforeCreate()
        {
               this.$store.dispatch('translation');
        },
        mounted(){
             this.$store.dispatch('translation');
            this.buildChart(this.chartdata,this.improved);
           
            
        },
        methods: {
            buildChart() {
                //  props: ['data','options'],
                this.renderChart({
                    labels: [this.get_text('min_allowed@gaugechart@l')],
                    datasets: [
                        {
                            backgroundColor: [
                                "#FF3333",

                            ],
                            data: [this.chartdata, (this.improved-this.chartdata)],
                        }
                    ]
                }, {
                    responsive: false, maintainAspectRatio: true, cutoutPercentage:70,

                })
            },
            get_text(textbit) {
                var transtext = this.$store.state.translations[this.$i18n.locale][textbit];
                if (transtext) {
                    return transtext;
                } else {
                    var spliteText = (textbit.split("@"));
                    if (spliteText.length > 2) {
                        var newTxt = (spliteText[0] + '@' + 'l');
                        var transtext1 = this.$store.state.translations[this.$i18n.locale][newTxt];
                        if (transtext1) {
                            return transtext1;
                        } else {
                            return "!" + textbit;
                        }
                    } else {
                        return "!" + textbit;
                    }
                }

            },
        },
        watch: {
            chartdata: {
                handler() {
                    this.chart_val = [this.chartdata,(this.improved-this.chartdata)];
                    this.buildChart(this.chartdata,this.improved)
                },
                deep: true
            }
        }
    }
</script>

