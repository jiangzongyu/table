<template>
    <div class="table-wrapper">
        <el-table
                :data="tableData"
                :style="tableStyle"
                :height="tableHeight"
                :stripe="stripe"
                :border="border"
                :row-class-name="rowClassName"
                :row-key="rowKey"
                :current-row-key="currentRowKey"
                :row-sytle="rowStyle"
                :heighlight-current-row="highlightCurrentRow"
                :show-header="showHeader"
                :empty-text="emptyText"
                :default-expand-all="defaultExpandAll"
                :default-sort="defaultSort"
                @current-change="currentChange"
                @select="select"
                @selection-change="selectChange"
                @select-all="selectAll"
                @cell-mouse-enter="cellMouseEnter"
                @cell-mouse-leave="cellMouseLeave"
                @cell-click="cellClick"
                @cell-dbclick="cellDbClick"
                @row-click="rowClick"
                @row-contextmenu="rowContextmenu"
                @row-dbclick="rowDbClick"
                @sort-change="sortChange"
                @filter-change="filterChange"
                @header-click="headerClick"
                @header-dradend="headerDragend"
                @expand="expand"
                :context="context"
                ref="elTable">
            <slot></slot>
        </el-table>
        <div class="table-loading"
             v-loading="loading"
             element-loading-text="加载中"></div>
        <el-row type="flex"
                class="row-bg"
                :justify="pagerPosition"
                v-if="showPagination">
            <el-pagination
                    ref="elPagination"
                    class="tal-table-el-pagination"
                    @size-change="pageSizeChange"
                    @current-change="pageCurrentChange"
                    :current-page="queryParams.page"
                    :page-sizes="pageSizes"
                    :page-size="queryParams.limit"
                    layout="total, sizes, prev, pager, next, jumper"
                    :total="tableDataTotal"
            ></el-pagination>
        </el-row>
    </div>
</template>

<script type="text/ecmascript-6">
    import Eltable from 'element-ui/lib/table'
    import Elrow from 'element-ui/lib/row'
    import Elcol from 'element-ui/lib/col'
    import { httpPost } from '~/utils/http'
    export default {
        name: 'tal-table',
        props: {
            queryUrl: {
                type: String,
                default: ''
            },
            deleteUrl: {
                type: String,
                default: ''
            },
            primaryKey: {
                type: String,
                default: 'id'
            },
            deleteConfirm: {
                type: Boolean,
                default: false
            },
            highlightCurrentRow: {
                type: Boolean,
                default: true
            },
            data: {
                type: Array,
                default() {
                    return []
                }
            },
            width: [String, Number],
            maxHeight: [String, Number],
            height: [String, Number],
            fit: {
                type: Boolean,
                default: true
            },
            stripe: {
                type: Boolean,
                default: false
            },
            border: {
                type: Boolean,
                default: false
            },
            rowClassName: [String, Function],
            rowKey: [String, Function],
            currentRowKey: [String, Number],
            rowStyle: [Function, Object],
            defaultExpandAll: {
                type: Boolean,
                default: false
            },
            defaultSort: Object,
            emptyText: {
                type: String,
                default: '暂无数据'
            },
            context: {
                type: Object,
                default: function () {
                    return this.$parent
                }
            },
            currentPage: {
                type: Number,
                default: 1
            },
            pageSize: {
                type: Number,
                default: 20
            },
            pageSizes: {
                type: Array,
                default: function () {
                    return [10, 20, 50, 100]
                }
            },
            pagerPosition: {
                type: String,
                default: 'end'
            },
            searchObject: {
                type: Object,
                default() {
                    return {}
                }
            },
            showHeader: {
                type: Boolean,
                default: true
            },
            showPagination: {
                type: Boolean,
                default: true
            },
            autoQuery: {
                type: Boolean,
                default: false
            }
        },
        components: {
            Eltable,
            Elrow,
            Elcol
        },
        data() {
            return {
                loading: false,
                tableStyle: {},
                tableHeight: 0,
                tableData: [],
                tableDataTotal: 0,
                rowSelected: [],
                selection: [],
                queryParams: {
                    limit: 20,
                    page: 1
                },
                lastRequest: null
            }
        },
        beforeMount() {
            if (this.height) {
                this.tableHeight = this.height
            } else {
                this.tableHeight = this.maxHeight
            }
        },
        mounted () {
            window.onresize = () => this.calcTableStyle()
            this.autoQuery && this.getDataRemote()
            this.$on('onRefresh', this.onRefresh)
            this.$on('onSearch', this.onSearch)
            this.$on('onDelete', this.onDelete)
        },
        watch: {
            queryUrl(newProps, oldProps) {
                this.getDataRemote()
            }
        },
        methods: {
            getDataRemote () {
                if (this.queryUrl) {
                    this.loading = true
                    let searchObject = {}
                    for (let key in this.searchObject) {
                        if (this.searchObject[key] !== '') {
                            searchObject[key] = this.searchObject[key]
                        }
                    }
                    httpPost(this.queryUrl, {
                        ...Object.assign(
                                {},
                                (this.showPagination ? this.queryParams : {}),
                                searchObject
                        )
                    }).then((res) => {
                                this.loading = false
                                if (res.status !== 200 || res.data.status !== 1) {
                                    this.tableData = []
                                    this.tableDataTotal = 0
                                    this.$message.error(
                                            res.body.info || '数据请求失败'
                                    )
                                } else {
                                    this.tableData = res.data.result.list || res.data.result.lists
                                    this.tableDataTotal = res.data.result.total
                                    if (this.tableDataTotal !== 1) {
                                        this.$nextTick(() => {
                                            this.calcTableStyle()
                                        })
                                    }
                                }
                                this.$emit('data-change', this.tableData, this.queryParams, this.tableDataTotal)
                            }, (res) => {
                                this.loading = false
                                this.tableData = []
                                this.$emit('data-change', this.tableData, this.queryParams.page, 0)
                                this.$message.error('数据获取失败....')
                            })
                            .catch((err) => {
                                console.log(err)
                            })
                }
            },
            pageSizeChange(newVal) {
                this.queryParams.page = 1
                this.queryParams.limit = newVal
                this.getDataRemote()
            },
            pageCurrentChange(newVal) {
                this.queryParams.page = newVal
                this.getDataRemote()
            },
            currentChange(row, oldCurrentRow) {
                this.rowSelected = row
                this.$emit('current-change', row, oldCurrentRow)
            },
            select(selection, row) {
                this.selection = selection
                this.$emit('select', selection, row)
            },
            selectAll(selection) {
                this.selection = selection
                this.$emit('select-all', selection)
            },
            selectChange(selection) {
                this.$emit('selection-change', selection)
            },
            cellMouseEnter (row, column, cell, event) {
                this.$emit('cell-mouse-enter', row, column, cell, event)
            },
            cellMouseLeave(row, column, cell, event) {
                this.$emit('cell-mouse-leave', row, column, cell, event)
            },
            cellClick(row, column, cell, event) {
                this.$emit('cell-click', row, column, cell, event)
            },
            cellDbClick(row, column, cell, evetn) {
                this.$emit('cell-dbclick', row, column, cell, event)
            },
            rowClick(row, event, column) {
                this.$emit('row-click', row, event, column)
            },
            rowContextmenu(row, event) {
                this.$emit('row-contextmenu', row, event)
            },
            rowDbClick(row, event, column) {
                this.$emit('row-dbclick', row, event, column)
            },
            headerClick(column, event) {
                this.$emit('header-click', column, event)
            },
            headerDragend(newWidth, oldWidth, column, event) {
                this.$emit('header-dragend', newWidth, oldWidth, column, event)
            },
            sortChange(o) {
                this.$emit('sort-change', o)
            },
            filterChange(filters) {
                this.$emit('filter-change', filters)
            },
            expand(row, expanded) {
                this.$emit('expand', row, expanded)
            },
            clearSelection() {
                this.$refs.elTable.clearSelection()
            },
            toggleRowSelection(row, selected) {
                this.$refs.elTable.toggleRowSelection(row, selected)
            },
            onRefresh() {
                this.getDataRemote()
            },
            onSearch() {
                this.queryParams.page = 1
                this.queryParams.limit = 20
                this.getDataRemote()
            },
            onDelete() {
                let params
                if (this.selection.length) {
                    params = []
                    this.selection.map((row) => {
                        params.push(row[this.primaryKey])
                    })
                } else {
                    params = this.rowSelected[this.primaryKey]
                }

                if (!params || params.length === 0) {
                    this.$message('请选择要删除的行！')
                    return false
                }
                if (this.deleteUrl) {
                    if (this.deleteConfirm) {
                        this.$confirm('将执行删除操作，是否继续？', '警告', {type: 'warning'})
                                .then(() => {
                                    this.execDelete(params)
                                }).catch((err) => { console.log(err) })
                    } else {
                        this.execDelete(params)
                    }
                } else {
                    this.$message.error('组件没有提供删除地址')
                }
            },
            execDelete(params) {
                let _params = {}
                _params[this.primaryKey] = params
                httpPost(this.deleteUrl, _params)
                        .then((res) => {
                            if (res.status !== 200 && res.body.code !== 0) {
                                this.$message.error(res.body.info || '删除失败')
                            } else {
                                this.$message.success('删除成功')
                                this.getDataRemote()
                            }
                        }, (res) => {
                            this.$message.error('服务器出现了一个问题.....')
                        })
            },
            calcTableStyle() {
                if (this.width) {
                    let _width
                    if (typeof this.width === 'string' && this.indexOf('%') !== -1) {
                        _width = this.width
                    } else {
                        _width = this.width + 'px'
                    }
                    this.tableStyle = {
                        width: _width
                    }
                }

                if (this.maxHeight && this.$refs.elTable) {
                    let el = this.$refs.elTable.$el
                    let elTableRect = el.getBoundingClientRect()
                    let headRect = el.querySelector('.el-table__header').getBoundingClientRect()
                    let bodyCls = this.tableDataTotal === 0 ? 'el-table__empty-block' : '.el-table__body'
                    let bodyRect = el.querySelector(bodyCls).getBoundingClientRect()
                    let elTableHeight = bodyRect.height + headRect.height
                    let elTableHeightWithPager = elTableHeight
                    let _h = 15
                    if (this.showPagination) {
                        let elPagination = this.$refs.elPagination.$el.getBoundingClientRect()
                        _h += elPagination.height + 5
                        elTableHeightWithPager += elPagination.height
                    }

                    if (this.maxHeight === 'auto') {
                        let bodyHeight = window.innerHeight
                        let _height = bodyHeight - elTableRect.top
                        if (_height < elTableHeightWithPager) {
                            this.tableHeight = _height - _h
                        } else if (elTableHeight === 0) {
                            let _h = this.tableDataTotal * 42
                            if (_h > _height) {
                                this.tableHeight = _height
                            } else {
                                this.tableHeight = _h + 42 - this.tableDataTotal
                            }
                        } else {
                            this.tableHeight = elTableHeight
                        }
                    } else {
                        if (this.maxHeight < elTableHeightWithPager) {
                            this.tableHeight = this.maxHeight
                        } else if (elTableHeight === 0) {
                            let _h = this.tabelDataTotal * 42
                            if (_h > this.maxHeight) {
                                this.tableHeight = this.maxHeight
                            } else {
                                this.tableHeight = _h + 42 - this.tableDataTotal
                            }
                        } else {
                            this.tableHeight = elTableHeight
                        }
                    }
                }
            }
        }
    }
</script>

<style scoped>
    .tal-table-el-pagination {
        margin-top: 10px
    }
    .table-loading {
        width: 100%;
        height: 100%;
        position: absolute!important;
        top: 0;
        left: 0;
        pointer-events: none;
    }
</style>
