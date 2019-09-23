<template>
    <div>
        <div class="operate-menus">
            <div class="menu-items menu-items-blue" @click="goRouter('business')">
                <div class="item-left">
                    <span v-if="navData.biz">{{ navData.biz }}</span>
                    <span v-if="!navData.biz">0</span>
                    <span>{{$t('业务总数')}}</span>
                </div>
                <div class="item-right item-right-left">
                    <i class="icon icon-cc-operate-biz first-icon"></i>
                </div>
            </div>
            <div class="menu-items menu-items-white" @click="goRouter('resource')">
                <div class="item-left">
                    <span v-if="navData.host">{{ navData.host }}</span>
                    <span v-if="!navData.host">0</span>
                    <span>{{$t('主机总数')}}</span>
                </div>
                <div class="item-right item-right-right">
                    <i class="icon icon-cc-host"></i>
                </div>
            </div>
            <div class="menu-items menu-items-blue" @click="goRouter('model')">
                <div class="item-left">
                    <span v-if="navData.model">{{ navData.model }}</span>
                    <span v-if="!navData.model">0</span>
                    <span>{{$t('模型总数')}}</span>
                </div>
                <div class="item-right item-right-left">
                    <i class="menu-icon icon-cc-operate-module"></i>
                </div>
            </div>
            <div class="menu-items menu-items-white">
                <div class="item-left">
                    <span v-if="navData.inst">{{ navData.inst }}</span>
                    <span v-if="!navData.inst">0</span>
                    <span>{{$t('实例总数')}}
                        <bk-popover :delay="300" :offset="10" placement="top" class="operate-pop">
                            <i class="menu-icon icon-cc-attribute"></i>
                            <span class="popover-content" slot="content">
                                {{tooltip}}
                            </span>
                        </bk-popover>
                    </span>
                </div>
                <div class="item-right item-right-right">
                    <i class="icon icon-cc-operate-exam first-icon"></i>
                </div>
            </div>
        </div>
        <div class="operation-top">
            <span class="operation-title">{{$t('主机统计')}}</span>
            <i class="icon icon-cc-operate-add" @click="openNew('add', 'host')">
                <div class="title-block">{{$t('添加主机图表')}}</div>
            </i>
        </div>
        <div v-for="(item, key) in hostData.disList"
            :key="item.report_type + item.config_id"
            :style="{ width: item.width + '%' }"
            class="operation-layout">
            <div class="chart-child">
                <div class="chart-title">
                    <span>{{item.name}}</span>
                    <div class="charts-options">
                        <i class="bk-icon icon-arrows-up icon-weight" :class="{ 'icon-disable': key === 0 }"
                            @click="moveChart('host', 'up', key, hostData.disList)">
                            <div class="title-block">{{$t('已置顶，无法上移')}}</div>
                        </i>
                        <i class="bk-icon icon-arrows-down icon-weight" :class="{ 'icon-disable': key === hostData.disList.length - 1 }"
                            @click="moveChart('host', 'down', key, hostData.disList)">
                            <div class="title-block">{{$t('已置底，无法下移')}}</div>
                        </i>
                        <i class="icon icon-cc-edit-shape"
                            @click="openNew('edit', 'host', item, key)"></i>
                        <i class="icon icon-cc-tips-close"
                            @click="deleteChart('host', key, hostData.disList, item)"></i>
                    </div>
                </div>
                <div class="chart-bottom">
                    <div class="operation-charts" :id="item.report_type + item.config_id"></div>
                </div>
                <div v-if="item.noData" class="null-data">
                    <span>{{$t('暂无数据')}}</span>
                </div>
                <div class="chart-date" v-if="item.showDate">
                    <bk-date-picker
                        :options="options"
                        @change="dateChange"
                        class="options-filter"
                        :type="'daterange'"
                        v-model="dateRange">
                    </bk-date-picker>
                </div>
            </div>
        </div>
        <div class="operation-top">
            <span class="operation-title">{{$t('实例统计')}}</span>
            <i class="icon icon-cc-operate-add" @click="openNew('add', 'inst')">
                <div class="title-block">{{$t('添加实例图表')}}</div>
            </i>
        </div>
        <div v-for="(item, key) in instData.disList"
            :key="item.report_type + item.config_id"
            :style="{ width: item.width + '%' }"
            class="operation-layout">
            <div class="chart-child">
                <div class="chart-title">
                    <span>{{item.name}}</span>
                    <div class="charts-options">
                        <i class="bk-icon icon-arrows-up icon-weight" :class="{ 'icon-disable': key === 0 }"
                            @click="moveChart('inst', 'up', key, instData.disList)">
                            <div class="title-block">{{$t('已置顶，无法上移')}}</div>
                        </i>
                        <i class="bk-icon icon-arrows-down icon-weight" :class="{ 'icon-disable': key === instData.disList.length - 1 }"
                            @click="moveChart('inst', 'down', key, instData.disList)">
                            <div class="title-block">{{$t('已置底，无法下移')}}</div>
                        </i>
                        <i class="icon icon-cc-edit-shape" @click="openNew('edit', 'inst', item, key)"></i>
                        <i class="icon icon-cc-tips-close"
                            @click="deleteChart('inst', key, instData.disList, item)"></i>
                    </div>
                </div>
                <div class="chart-bottom">
                    <div class="operation-charts" :id="item.report_type + item.config_id"></div>
                </div>
                <div v-if="item.noData" class="null-data">
                    <span>{{$t('暂无数据')}}</span>
                </div>
            </div>
        </div>
        <v-detail v-if="isShow"
            :open-type="editType.openType"
            :host-type="editType.hostType"
            :chart-data="newChart"
            @transData="saveData"
            @closeChart="cancelData">
        </v-detail>
    </div>
</template>

<script>
    import Plotly from 'plotly.js'
    import { mapActions } from 'vuex'
    import vDetail from './chart-detail'
    export default {
        name: 'index',
        components: {
            vDetail
        },
        data () {
            return {
                tooltip: this.$t('不包含业务、主机模型及实例'),
                isShow: false,
                newChart: {},
                editType: {
                    openType: 'add',
                    hostType: 'host',
                    index: ''
                },
                hostData: {
                    disList: []
                },
                instData: {
                    disList: []
                },
                navData: {
                    biz: '',
                    host: '',
                    module: '',
                    inst: '',
                    model: ''
                },
                dateRange: [],
                dateChart: {},
                maxRange: [],
                options: {
                    disabledDate (date) {
                        const today = new Date()
                        return today < date
                    }
                }
            }
        },
        created () {
            this.$store.commit('setHeaderTitle', this.$t('统计报表'))
            this.getChartList()
        },
        methods: {
            ...mapActions('operationChart', [
                'getCountedCharts',
                'getCountedChartsData',
                'deleteOperationChart',
                'updateChartPosition'
            ]),
            async getChartList () {
                const res = await this.getCountedCharts({})
                this.hostData.disList = res.info.host
                this.instData.disList = res.info.inst
                res.info.nav.forEach(item => {
                    this.getNavData(item, 'nav')
                })
                this.hostData.disList.forEach((item) => {
                    this.getNavData(item, 'host')
                })
                this.instData.disList.forEach((item) => {
                    this.getNavData(item, 'inst')
                })
            },
            async getNavData (item, type) {
                const res = await this.getCountedChartsData({
                    params: {
                        config_id: item.config_id
                    },
                    config: {
                        globalError: false
                    }
                })
                if (type === 'nav') {
                    res.forEach(items => {
                        this.navData[items.id] = items.count
                    })
                } else {
                    if (JSON.stringify(res) === '{}' || JSON.stringify(res) === '[]' || res === null) {
                        item.hasData = false
                        item.data = {
                            data: [{
                                labels: [],
                                type: item.chart_type,
                                values: [],
                                x: [],
                                y: []
                            }]
                        }
                    } else {
                        item.hasData = true
                        item.data = await this.dataDeal(item, res)
                    }
                    this.drawCharts(item)
                }
            },
            dataDeal (data, res) {
                const returnData = {
                    data: [],
                    minTime: '',
                    maxTime: '',
                    maxRange: 0
                }
                const hoverlabel = {
                    bgcolor: '#fff',
                    bordercolor: '#DCDEE5',
                    font: {
                        color: '#63656E',
                        size: '12'
                    }
                }
                const marker = {
                    size: 16,
                    color: []
                }
                if (res && Array.isArray(res)) {
                    const content = {
                        labels: [],
                        values: [],
                        x: [],
                        y: [],
                        type: data.chart_type,
                        hoverlabel: hoverlabel,
                        marker: marker,
                        mode: 'markers',
                        textinfo: 'none',
                        hole: 0.7,
                        pull: 0.01,
                        width: 0.1
                    }
                    let zeroList = 0
                    res.forEach(item => {
                        if (item.count === 0) zeroList++
                        if (data.chart_type === 'pie') {
                            content.labels.push(item.id)
                            content.values.push(item.count)
                        } else {
                            const color = '#3A84FF'
                            content.marker.color.push(color)
                            content.x.push(item.id)
                            content.y.push(item.count)
                        }
                    })
                    if (zeroList === res.length) data.hasData = false
                    if (data.chart_type !== 'pie') content.width = (0.015 * data.x_axis_count * (data.width === '50' ? 2 : 1))
                    returnData.data.push(content)
                } else if (res && !Array.isArray(res)) {
                    const barModel = {
                        chart: {
                            delete: {
                                name: 'delete',
                                type: 'bar',
                                y: [],
                                color: '#3A84FF',
                                marker: this.$tools.clone(marker),
                                hoverinfo: 'y+name',
                                hoverlabel: hoverlabel
                            },
                            update: {
                                name: 'update',
                                type: 'bar',
                                y: [],
                                color: '#38C1E2',
                                marker: this.$tools.clone(marker),
                                hoverinfo: 'y+name',
                                hoverlabel: hoverlabel
                            },
                            create: {
                                name: 'create',
                                type: 'bar',
                                y: [],
                                color: '#59D178',
                                marker: this.$tools.clone(marker),
                                hoverinfo: 'y+name',
                                hoverlabel: hoverlabel
                            }
                        },
                        x: [],
                        isX: false
                    }
                    for (const item in res) {
                        const content = {
                            name: item,
                            x: [],
                            y: [],
                            hoverlabel: hoverlabel,
                            textinfo: 'none'
                        }
                        if (Array.isArray(res[item])) {
                            barModel.isX = false
                            res[item].forEach(child => {
                                const time = this.$tools.formatTime(child.id, 'YYYY-MM-DD')
                                returnData.minTime = returnData.minTime === '' ? time : returnData.minTime
                                returnData.maxTime = time > returnData.maxTime ? time : returnData.maxTime
                                returnData.minTime = time < returnData.minTime ? time : returnData.minTime
                                content.x.push(time)
                                content.y.push(child.count)
                                content.mode = 'line'
                            })
                            returnData.data.push(content)
                        } else {
                            barModel.x.push(item)
                            barModel.isX = true
                            for (const chartItem in barModel.chart) {
                                barModel.chart[chartItem].y.push(res[item][chartItem])
                                barModel.chart[chartItem].marker.color.push(barModel.chart[chartItem].color)
                            }
                        }
                    }
                    if (barModel.isX) {
                        for (const chartItem in barModel.chart) {
                            barModel.chart[chartItem].x = barModel.x
                            returnData.data.push(barModel.chart[chartItem])
                        }
                    }
                }
                if (data.report_type === 'host_change_biz_chart') this.maxRange = [returnData.minTime, returnData.maxTime]
                return returnData
            },
            drawCharts (item) {
                const layConfig = {
                    type: 'category',
                    nticks: item.x_axis_count,
                    range: [-0.5, item.x_axis_count - 0.5],
                    fixRange: item.x_axis_count >= item.data.data[0].x.length,
                    colorway: ['#3A84FF', '#A3C5FD', '#59D178', '#94F5A4', '#38C1E2', '#A1EDFF', '#4159F9', '#7888F0', '#EFAF4B', '#FEDB89', '#FF5656', '#FD9C9C'],
                    legend: {
                        orientation: 'h',
                        xanchor: 'auto',
                        yanchor: 'bottom',
                        valign: 'bottom',
                        x: 0.48,
                        y: -0.5
                    }
                }
                const myDiv = item.report_type + item.config_id
                const data = item.data.data
                if (!item.hasData) this.$set(item, 'noData', true)
                if (item.chart_type === 'pie') {
                    layConfig.legend = {
                        xanchor: 'auto',
                        yanchor: 'auto',
                        valign: 'right',
                        y: 0.5
                    }
                }
                if (item.report_type === 'host_change_biz_chart') {
                    if (this.dateRange.length !== 0) {
                        item.data.minTime = this.dateRange[0]
                        item.data.maxTime = this.dateRange[1]
                    } else {
                        this.dateRange = [item.data.minTime, item.data.maxTime]
                    }
                    layConfig.type = 'date'
                    layConfig.range = [item.data.minTime, item.data.maxTime]
                    this.dateChart = item
                    this.$set(item, 'showDate', true)
                    layConfig.fixRange = (this.maxRange[0] >= item.data.minTime) && (this.maxRange[1] <= item.data.maxTime)
                    layConfig.nticks = item.x_axis_count % 2 === 0 ? item.x_axis_count : (item.x_axis_count + 1)
                    layConfig.colorway = ['#3A84FF', '#59D178', '#38C1E2', '#4159F9', '#EFAF4B', '#FF5656', '#904DF7', '#CA78F0', '#B9E145', '#F6E354']
                }
                const layout = {
                    barmode: 'stack',
                    height: 250,
                    margin: {
                        l: 50,
                        r: 50,
                        t: 30,
                        b: 50
                    },
                    yaxis: {
                        fixedrange: true,
                        rangemode: 'tozero',
                        zeroline: false
                    },
                    xaxis: {
                        showline: true,
                        linecolor: '#BFBFBF',
                        type: layConfig.type,
                        nticks: item.x_axis_count,
                        range: layConfig.range,
                        showgrid: false,
                        autorange: false,
                        tickformat: '%Y-%m-%d',
                        fixedrange: layConfig.fixRange
                    },
                    bargap: 0.1,
                    bargroupgap: 1.0044 - (0.0401 * item.x_axis_count),
                    dragmode: 'pan',
                    colorway: layConfig.colorway,
                    legend: layConfig.legend
                }
                if (item.report_type !== 'model_inst_change_chart') layout.hovermode = 'closest'
                const options = {
                    displaylogo: false,
                    displayModeBar: false,
                    responsive: true
                }
                if (this.editType.openType === 'edit') {
                    Plotly.purge(myDiv)
                    setTimeout(() => {
                        Plotly.newPlot(myDiv, data, layout, options)
                        if (item.data.data[0].mode !== 'line') this.hoverConfig(document.getElementById(myDiv), myDiv)
                    }, 100)
                } else {
                    Plotly.newPlot(myDiv, data, layout, options)
                    if (item.data.data[0].mode !== 'line') this.hoverConfig(document.getElementById(myDiv), myDiv)
                }
            },
            hoverConfig (myPlot, myDiv) {
                myPlot.on('plotly_hover', (data) => {
                    const colors = ['#A3C5FD', '#B0E7F4', '#BDEDC9']
                    this.reStyle(myDiv, data, colors)
                })
                myPlot.on('plotly_unhover', (data) => {
                    const colors = ['#3A84FF', '#38C1E2', '#59D178']
                    this.reStyle(myDiv, data, colors)
                })
            },
            reStyle (myDiv, data, color) {
                let pn = ''
                let tn = ''
                let colors = []
                for (let i = 0; i < data.points.length; i++) {
                    pn = data.points[i].pointNumber
                    tn = data.points[i].curveNumber
                    colors = data.points[i].data.marker.color
                }
                if (data.points.length === 1) {
                    colors[pn] = color[0]
                    const update = {
                        'marker': {
                            color: colors
                        }
                    }
                    Plotly.restyle(myDiv, update, [tn])
                } else {
                    data.points[0].data.marker.color[pn] = color[0]
                    data.points[1].data.marker.color[pn] = color[1]
                    data.points[2].data.marker.color[pn] = color[2]
                    const update = {
                        data: {
                            maker: {
                                color: [data.points[0].data.marker.color, data.points[1].data.marker.color, data.points[2].data.marker.color]
                            }
                        }
                    }
                    Plotly.restyle(myDiv, update, [0, 2])
                }
            },
            moveChart (type, dire, key, list) {
                if (dire === 'up' && key !== 0) {
                    list[key] = list.splice(key - 1, 1, list[key])[0]
                } else if (dire === 'down' && key !== list.length - 1) {
                    list[key + 1] = list.splice(key, 1, list[key + 1])[0]
                }
                if (type === 'host') this.hostData.disList = list
                else this.instData.disList = list
                this.updatePosition()
            },
            deleteChart (type, key, list, item) {
                this.$bkInfo({
                    title: this.$tc('Operation["是否确认删除"]'),
                    subTitle: '确定要删除【' + item.name + '】',
                    extCls: 'bk-dialog-sub-header-center',
                    confirmFn: () => {
                        list.splice(key, 1)
                        if (type === 'host') this.hostData.disList = list
                        else this.instData.disList = list
                        this.deleteOperationChart({
                            id: item.config_id
                        })
                        this.updatePosition()
                    }
                })
            },
            async openNew (type, host, data, key) {
                this.editType.hostType = host
                this.editType.key = key
                this.editType.openType = type
                if (type === 'edit') {
                    this.newChart = this.$tools.clone(data)
                    this.newChart.title = data.name
                    this.newChart.bk_obj_id = data.bk_obj_id ? data.bk_obj_id : host
                } else {
                    this.newChart = {
                        report_type: 'custom',
                        name: '',
                        config_id: null,
                        bk_obj_id: host,
                        chart_type: 'pie',
                        field: '',
                        width: '50',
                        x_axis_count: 10
                    }
                }
                this.isShow = true
            },
            async saveData (data) {
                let editList = []
                if (this.editType.hostType === 'host') editList = this.hostData.disList
                else editList = this.instData.disList
                if (this.editType.openType === 'add') {
                    editList.push(data)
                } else {
                    editList[this.editType.key] = data
                }
                this.isShow = false
                await this.getNavData(data, this.editType.hostType)
                this.updatePosition()
                this.newChart = {}
            },
            cancelData () {
                this.isShow = false
                this.newChart = {}
            },
            updatePosition () {
                const data = {
                    'host': [],
                    'inst': []
                }
                this.hostData.disList.forEach(item => {
                    data.host.push(item.config_id)
                })
                this.instData.disList.forEach(item => {
                    data.inst.push(item.config_id)
                })
                this.updateChartPosition({
                    params: {
                        position: data
                    }
                })
            },
            goRouter (route) {
                this.$router.push(route)
            },
            dateChange (date) {
                this.dateChart.data.maxTime = date[1]
                this.dateChart.data.minTime = date[0]
                this.editType.openType = 'edit'
                this.drawCharts(this.dateChart)
            }
        }
    }
</script>

<style scoped lang="scss">
    .views-layout {
        padding: 10px!important;
        background:rgba(250,251,253,1);
    }
    *{
        margin: 0;
        padding: 0
    }
    .title-block{
        display: none;
        position: absolute;
        width: max-content;
        top: -140%;
        left: 0;
        color: #fff;
        height: 24px;
        padding: 0 5px;
        line-height: 22px;
        font-size: 12px;
        border-radius: 3px;
        transform: translate(-37%);
        background: rgba(0,0,0,0.8);
    }
    .title-block:before{
        position: absolute;
        content: "";
        width: 0;
        height: 0;
        left: 50%;
        transform: translate(-5px);
        bottom: -5px;
        border-left: 5px solid transparent;
        border-top: 5px solid rgba(0,0,0,0.8);
        border-right: 5px solid transparent;
    }
    .operation-top{
        padding: 10px 15px;
        margin-top: 15px;
        .operation-title{
            display: inline-block;
            font-size: 16px;
            font-weight: bold;
            color: #313238;
            vertical-align: middle;
        }
        i{
            position: relative;
            color: #3A84FF;
            font-size: 22px;
            cursor: pointer;
            margin-left: 5px;
            vertical-align: middle;
        }
        i:hover{
            background:rgba(225,236,255,1);
            .title-block {
                display: block;
            }
        }
    }
    .operate-menus{
        height: 105px;
        display: flex;
        width: 100%;
        .menu-items-blue{
            background: linear-gradient(165deg,rgba(108,186,255,1) 0%,rgba(58,132,255,1) 100%);
            &:hover{
                 box-shadow:0px 2px 6px 0px rgba(58,132,255,1);
            }
        }
        .menu-items-white{
            background: linear-gradient(165deg,rgba(77,212,240,1) 0%,rgba(27,167,207,1) 100%);
            &:hover{
                 box-shadow:0px 2px 6px 0px rgba(27,167,207,1);
            }
        }
        .menu-items{
            display: flex;
            align-items: center;
            width: 25%;
            height: 100%;
            margin: 10px;
            float: left;
            cursor: pointer;
            box-shadow: 0 2px 4px 0 rgba(220,222,229,1);
            border-radius: 4px;
            position: relative;
            .item-left{
                width: 60px;
                height: 45px;
                float: left;
                margin-left: 26px;
                line-height: 45px;
                span{
                    display: block;
                    text-align: left;
                    &:first-child{
                         width: 29px;
                         height: 29px;
                         font-size: 24px;
                         font-weight: bold;
                         color: rgba(255,255,255,1);
                         line-height: 29px;
                    }
                    &:last-child{
                         height: 16px;
                         width: 90px;
                         font-size: 12px;
                         font-weight: bold;
                         color: rgba(255,255,255,1);
                         line-height: 16px;
                         i{
                             display: inline-block;
                             font-size: 16px;
                             margin-left: 5px;
                         }
                    }
                    .operate-pop{
                        vertical-align: bottom;
                    }
                }
            }
            .item-right-left{
                background-color: #377AE9;
            }
            .item-right-right{
                background-color: #19A2CA;
            }
            .item-right{
                 display: flex;
                 justify-content: center;
                 align-items: center;
                 height: 60px;
                 width: 60px;
                 position: absolute;
                 right: 35px;
                 border-radius: 50%;
               i{
                   color: white;
                   font-size: 24px;
               }
               .first-icon {
                   font-size: 20px;
               }
            }
        }
    }
    .operation-layout{
        display: inline-block;
        position: relative;
        padding: 10px 10px;
        .chart-child{
             background: rgba(255,255,255,1);
             border-radius: 2px;
             border: 1px solid #DCDEE5;
             position: relative;
             width: 100%;
             .operation-charts{
                width: 100%;
             }
             &:hover {
                  box-shadow:0 2px 6px 0 rgba(220,222,229,1);
             }
        }
        .chart-child:hover{
            .charts-options{
                display: inline-block;
            }
        }
        .chart-bottom{
            width: 100%;
            height: 100%;
            padding: 0 10px 10px;
        }
        .chart-title{
            display: block;
            height: 50px;
            border-bottom: 1px solid #F0F1F5;
            .charts-options{
                display: none;
                margin-left: 15px;
                line-height: 48px;
                color: #3A84FF;
                i{
                    box-sizing: border-box;
                    cursor: pointer;
                    margin-left: 3px;
                    border: 1px solid rgba(99,101,110,0);
                    padding: 1px;
                    &:hover{
                         color: #2D6ACF
                    }
                }
            }
            span{
                line-height: 50px;
                font-size: 14px;
                font-weight: bold;
                color: rgba(99,101,110,1);
                margin-left: 21px;
            }
        }
        .chart-date{
            line-height: 48px;
            position: absolute;
            right: 25px;
            top: 0;
            width: 250px;
        }
        .disable-title {
            display: none;
        }
        .icon-disable{
            color: #DCDEE5!important;
            position: relative;
            &:hover{
                .title-block {
                    top: -165%;
                    transform: translate(-42%);
                    display: block;
                }
             }
        }
        .icon-weight{
            padding: 0!important;
            font-size: 18px;
            font-weight: bold;
        }
        .null-data {
            position: absolute;
            top: 50%;
            width: 100%;
            text-align: center;
            height: 19px;
            span {
                margin: 0 auto;
                display: block;
                font-size: 14px;
                color: rgba(151,155,165,1);
                line-height: 19px;
            }
        }
    }
</style>