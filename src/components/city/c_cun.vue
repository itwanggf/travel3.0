<template>
	<div class="public cun " @click.stop='isFilter=false'>
		<div class="main">
			<div class="main_content">
				<div class="filter" @mouseenter='isFilter=true' @click.stop='isFilter=true'>
					<el-form :inline="true" :model="filter" class="demo-form-inline">
						<el-form-item label="归属区县">
							<el-select v-model="filter.countryId" clearable placeholder="全部" @change="handleCountryId">
								<el-option v-for="item in countyOps" :key="item.id" :label="item.orgName" :value="item.id">
								</el-option>
							</el-select>
						</el-form-item>
						<el-form-item label="归属乡镇">
							<el-select v-model="filter.townId" clearable placeholder="全部">
								<el-option v-for="item in villageOps" :key="item.id" :label="item.orgName" :value="item.id">
								</el-option>
							</el-select>
						</el-form-item>
						<el-form-item label="村名">
							<el-input v-model="filter.user" clearable placeholder="村名"></el-input>
						</el-form-item>
					</el-form>
					<div class="filter_btn">
						<div class="btn btn1" @click.stop="addquery">查询</div>
						<div class="btn btn3" @click="exports">批量导出</div>
					</div>
				</div>
				<div class="table">
					<el-table ref="table" :data="tableData" tooltip-effect="dark" style="width: 99%;" height="100%" header-cell-class-name="tableTou" cell-class-name="tableRow" @selection-change="handleSelectionChange">
						<el-table-column type="index" :index="indexMethod" label="序号" align="center" width="100px">
						</el-table-column>
						<el-table-column prop="villageName" label="村名" align="center" show-overflow-tooltip>
						</el-table-column>
						<el-table-column prop="city" align="center" label="归属市州" show-overflow-tooltip>
						</el-table-column>
						<el-table-column prop="country" align="center" label="归属区县" show-overflow-tooltip>
						</el-table-column>
						<el-table-column prop="town" align="center" label="归属乡镇" show-overflow-tooltip>
						</el-table-column>
						<el-table-column prop="villageHeaderName" align="center" label="村长姓名" show-overflow-tooltip>
						</el-table-column>
						<el-table-column prop="villageHeaderPhone" align="center" label="村长联系电话" show-overflow-tooltip>
						</el-table-column>
						<el-table-column prop="villageSecretaryName" align="center" label="村支书姓名" show-overflow-tooltip>
						</el-table-column>
						<el-table-column prop="villageSecretaryPhone" align="center" label="村支书联系电话" width="130" show-overflow-tooltip>
						</el-table-column>
						<el-table-column prop="residentSecretaryName" align="center" label="驻村第一书记姓名" width="130" show-overflow-tooltip>
						</el-table-column>
						<el-table-column prop="residentSecretaryPhone" align="center" label="驻村第一书记联系电话" width="130" show-overflow-tooltip>
						</el-table-column>
						<el-table-column prop="updateTime" align="center" label="更新日期" show-overflow-tooltip>
							<template slot-scope="scope">
								{{!scope.row.updateTime ? '' : scope.row.updateTime.slice(0, 10)}}
							</template>
						</el-table-column>
					</el-table>

				</div>
				<div class="fenye">
					<div class="fenye_title">共 {{total}} 条数据 每页 {{pageSize}} 条数据 共 {{Math.floor(total/pageSize) + (total%pageSize != 0 ? 1 : 0)}} 页</div>
					<el-pagination :current-page.sync="currentPage" @current-change="handleCurrentChange" :page-size="pageSize" layout="prev, pager, next, jumper" :total="total">
					</el-pagination>
				</div>
			</div>
		</div>
		<div class="footer">
			<my-footer></my-footer>
		</div>
	</div>
</template>

<script>
import fetch from "@/assets/tools/fetch"
import cun from "@/assets/api/cun"
import footer from '@/components/com/footer'
import allApi from "@/assets/api/user"
export default {
	data() {
		return {
			countyOps: [],
			villageOps: [],
			city: '',
			optionscity: [{}],
			isFilter: false, // 是否展开筛选
			filter: {
				countryId: '',
				townId: '',
				user: ''
			},
			pageSize: 10,
			total: 0,
			currentPage: 1,

			tableData: [
			],
			multipleSelection: [],
		}
	},
	created() {
		this.getData();
		this.getFilter();
	},
	components: { 'my-footer': footer },
	methods: {
		indexMethod(index) { return index + 1 + (this.currentPage - 1) * this.pageSize; },
		toggleSelection(rows) {
			if (rows) {
				rows.forEach(row => {
					this.$refs.multipleTable.toggleRowSelection(row);
				});
			} else {
				this.$refs.multipleTable.clearSelection();
			}
		},
		handleSelectionChange(val) {
			this.multipleSelection = val;
		},
		handleCurrentChange() {
			this.getData();
		},
		// 查询
		addquery() {
			this.currentPage = 1;
			this.getData()
			this.isFilter = false;

		},

		// 获取村信息列表
		getData() {
			let par = {
				pageSize: this.pageSize,
				pageNumber: this.currentPage,
				villageName: this.filter.user,
				town: this.filter.townId,
				country: this.filter.countryId,
				city: ''
			}
			fetch.get(cun.cunxingz, par, res => {
				if (res.code == 200) {
					this.tableData = res.data;
					this.total = res.total;


				} else {
					this.tableData = [];
					this.total = 0

				}

			}, { target: '.el-table__body-wrapper' });
		},
		getFilter() {
			let par = {

			}
			fetch.get(allApi.getDivsion, par, res => {
				if (res.code === 200) {
					this.countyOps = res.data
				}
			})
		},
		handleCountryId(v) {
			this.filter.townId = '';
			this.filter.villageId = '';
			this.villageOps = [];
			if (!v) {
				return
			};
			let par = {
				orgId: this.filter.countryId,
				orgLevel: 4
			}
			fetch.get(allApi.getDivsion, par, res => {
				if (res.code === 200) {
					this.villageOps = res.data
				}
			})
		},
		// 导出
		exports() {
			let { user, townId, countryId, city } = this.filter;
			let par = {
				villageName: this.filter.user,
				town: this.filter.townId,
				country: this.filter.countryId,
				city: this.filter.city
			}
			// return;
			fetch.export(cun.dcvillage, par, res => {
				if (res.code === 200) {
					window.location = res.href
				}
			}, { target: '.btn3', loadingText: '正在导出' })
		},
	}
}

</script>

<style scoped>
.public {
  height: 100%;
  position: relative;
}
.footer {
  height: 60px;
  position: absolute;
  bottom: 0;
  left: 0;
  right: 0;
}
.main {
  position: absolute;
  top: 0;
  bottom: 60px;
  left: 0;
  right: 0;
  background: #eff2f7;
  padding: 20px 40px 0px 20px;
}
.main_content {
  height: 100%;
  background: rgba(255, 255, 255, 1);
  display: flex;
  flex-direction: column;
  border-radius: 3px;
  box-shadow: 0px 0px 120px 0px rgba(31, 32, 41, 0.05);
  overflow: auto;
}
.filter {
  height: 74px;
  display: flex;
  justify-content: start;
  padding-left: 30px;
  align-items: center;
}
.isFilter {
  width: 190px;
  height: 50px;
  background: rgba(244, 244, 244, 1);
  border-radius: 10px;
  display: flex;
  justify-content: center;
  align-items: center;
  cursor: pointer;
  font-size: 18px;
  font-weight: 400;
  color: rgba(61, 97, 150, 1);
}
.rotate {
  transform: rotate(180deg);
}
.isFilter > img {
  margin-right: 10px;
  transition: all 0.3s;
}
.filterBox {
  position: absolute;
  border-top: 1px solid rgba(238, 238, 238, 1);
  padding: 36px 50px 46px 50px;
  height: auto;
  z-index: 999;
  background-color: #fff;
  top: 74px;
  left: 0;
  right: 0;
  opacity: 0.99;
  box-shadow: 0px 10px 40px rgba(0, 0, 0, 0.14);
}
.filter_btn {
  display: flex;
  align-items: center;
}
.filter_btn > div {
  margin-left: 30px;
}
.table {
  flex: 1;
}
</style>