<template>
    <section>
        <ul class="template-list clearfix"
            v-bkloading="{ isLoading: $loading('getServiceTemplate') }"
            :class="{ 'is-loading': $loading('getServiceTemplate') }">
            <template v-if="templates.length">
                <li class="template-item fl clearfix"
                    v-for="(template, index) in templates"
                    :class="{
                        'is-selected': localSelected.includes(template.id),
                        'is-middle': index % 3 === 1
                    }"
                    :key="template.id"
                    @click="handleClick(template)">
                    <i class="select-icon bk-icon icon-check-circle-shape fr"></i>
                    <span class="template-name" :title="template.name">{{template.name}}</span>
                </li>
            </template>
            <li class="template-empty" v-else>
                <div class="empty-content">
                    <img class="empty-image" src="../../../assets/images/empty-content.png">
                    <i18n class="empty-tips" path="无服务模板提示">
                        <a class="empty-link" href="javascript:void(0)" place="link" @click="handleLinkClick">{{$t('跳转添加')}}</a>
                    </i18n>
                </div>
            </li>
        </ul>
    </section>
</template>

<script>
    export default {
        name: 'serviceTemplateSelector',
        props: {
            selected: {
                type: Array,
                default () {
                    return []
                }
            }
        },
        data () {
            return {
                templates: [],
                localSelected: [...this.selected]
            }
        },
        created () {
            this.getTemplates()
        },
        methods: {
            async getTemplates () {
                try {
                    const data = await this.$store.dispatch('serviceTemplate/searchServiceTemplate', {
                        params: this.$injectMetadata({}),
                        config: {
                            requestId: 'getServiceTemplate'
                        }
                    })
                    this.templates = data.info.map(datum => datum.service_template).sort((A, B) => {
                        return A.name.localeCompare(B.name, 'zh-Hans-CN', { sensitivity: 'accent' })
                    }).sort((A, B) => {
                        const weightA = this.selected.includes(A.id) ? 1 : 0
                        const weightB = this.selected.includes(B.id) ? 1 : 0
                        return weightB - weightA
                    })
                } catch (e) {
                    console.error(e)
                    this.templates = []
                }
            },
            handleClick (template) {
                const index = this.localSelected.indexOf(template.id)
                if (index > -1) {
                    this.localSelected.splice(index, 1)
                } else {
                    this.localSelected.push(template.id)
                }
            },
            getSelectedServices () {
                return this.localSelected.map(id => this.templates.find(template => template.id === id))
            },
            handleLinkClick () {
                this.$router.push({
                    name: 'operationalTemplate'
                })
            }
        }
    }
</script>

<style lang="scss" scoped>
    .template-list {
        max-height: 340px;
        @include scrollbar-y;
        &.is-loading {
            min-height: 144px;
        }
        .template-item {
            width: calc((100% - 20px) / 3);
            height: 32px;
            margin: 0 0 16px 0;
            padding: 0 6px 0 10px;
            line-height: 30px;
            border-radius: 2px;
            border: 1px solid #DCDEE5;
            color: #63656E;
            cursor: pointer;
            &.is-middle {
                margin: 0 10px 16px;
            }
            &.is-selected {
                background-color: #E1ECFF;
                .select-icon {
                    font-size: 18px;
                    border: none;
                    border-radius: initial;
                    background-color: initial;
                    color: #3A84FF;
                }
            }
            .select-icon {
                width: 18px;
                height: 18px;
                font-size: 0px;
                margin: 6px 0;
                color: #fff;
                background-color: #fff;
                border-radius: 50%;
                border: 1px solid #979BA5;
            }
        }
    }
    .template-empty {
        height: 280px;
        &:before {
            content: "";
            height: 100%;
            width: 0;
            @include inlineBlock;
        }
        .empty-content {
            width: 100%;
            @include inlineBlock;
            .empty-image {
                display: block;
                margin: 0 auto;
            }
            .empty-tips {
                display: block;
                text-align: center;
            }
            .empty-link {
                color: #3A84FF;
            }
        }
    }
</style>