<template>
  <el-table :data="$store.state.employee.employeesList" border stripe style="width: 100%">
    <el-table-column prop="id" label="ID" min-width="100"></el-table-column>
    :data="$store.state.employee.employeesList"
    <el-table-column prop="name" label="姓名" min-width="150">
      <template slot-scope="scope">
        <el-input v-if="isEdited[scope.$index]" size="small" v-model="editTextByName"></el-input>
        <span v-else>{{scope.row.name}}</span>
      </template>
    </el-table-column>

    <el-table-column prop="mail" label="Email" min-width="250">
      <template slot-scope="scope">
        <el-input v-if="isEdited[scope.$index]" size="small" v-model="editTextByMail"></el-input>
        <span v-else>{{scope.row.mail}}</span>
      </template>
    </el-table-column>

    <el-table-column prop="telephone" label="电话号码" min-width="200">
      <template slot-scope="scope">
        <el-input v-if="isEdited[scope.$index]" size="small" v-model="editTextByTelephone"></el-input>
        <span v-else>{{scope.row.telephone}}</span>
      </template>
    </el-table-column>

    <el-table-column prop="role" label="角色" min-width="150">
      <template slot-scope="scope">
        <el-input v-if="isEdited[scope.$index]" size="small" v-model="editTextByRole"></el-input>
        <span v-else>{{scope.row.role}}</span>
      </template>
    </el-table-column>

    <el-table-column prop="status" label="状态" min-width="150">
      <template slot-scope="scope">
        <el-input v-if="isEdited[scope.$index]" size="small" v-model="editTextByStatus"></el-input>
        <span v-else>{{scope.row.status}}</span>
      </template>
    </el-table-column>

    <el-table-column label="操作" min-width="150">
      <template slot-scope="scope">
        <el-button
          size="mini"
          type="primary"
          @click="handleEdit(scope.$index, scope.row)"
          :disabled="isModifyButtonDisabledArray[scope.$index]"
        >{{editButtonName[scope.$index]}}</el-button>
        <el-button
          size="mini"
          type="danger"
          @click="freeze(scope.$index, scope.row)"
        >{{operateButtonNameArray[scope.$index]}}</el-button>
      </template>
    </el-table-column>
  </el-table>
</template>

<script>
import { TABLE_BUTTON_TYPE } from "../../config/const-values";
import {
  UPDATE_EMPLOYEE,
  GET_EMPLOYEES_LIST
} from "../../store/const/employee-const";
import { employeeRole, employeeStatus } from "../../config/util";
export default {
  name: "EmployeesTable",
  data: function() {
    return {
      isModifyButtonDisabledArray: [],
      operateButtonNameArray: [],
      editTextByName: "",
      editTextByMail: "",
      editTextByTelephone: "",
      editTextByRole: "",
      editTextByStatus: "",
      isEdited: [],
      editButtonName: []
    };
  },
  mounted() {
    this.$store
      .dispatch(GET_EMPLOYEES_LIST)
      .then(() => {
        const employeeLength = this.$store.state.employee.employeesList.length;
        for (let i = 0; i < employeeLength; i++) {
          this.isModifyButtonDisabledArray.push(false);
          this.operateButtonNameArray.push("冻结");
          this.isEdited.push(false);
          this.editButtonName.push(TABLE_BUTTON_TYPE[0]);
        }
      })
      .catch(() => {});
  },
  updated() {
    const employeeLength = this.$store.state.employee.employeesList.length;
    this.isModifyButtonDisabledArray.splice(0);
    this.operateButtonNameArray.splice(0);
    this.isEdited.splice(0);
    this.editButtonName.splice(0);
    for (let i = 0; i < employeeLength; i++) {
      this.isModifyButtonDisabledArray.push(false);
      this.operateButtonNameArray.push("冻结");
      this.isEdited.push(false);
      this.editButtonName.push(TABLE_BUTTON_TYPE[0]);
    }
  },
  methods: {
    freeze: function(index, row) {
      if (this.operateButtonNameArray[index] === "冻结") {
        this.$confirm("此操作将冻结该员工，是否继续?", "提示", {
          confirmButtonText: "确定",
          cancelButtonText: "取消",
          type: "warning"
        })
          .then(() => {
            const updateEmployee = {
              
            }
            row.role = employeeRole[row.role]
            row.status = employeeStatus.FREEZED
            this.$store
              .dispatch(UPDATE_EMPLOYEE, row)
              .then(response => {
                this.isModifyButtonDisabledArray.splice(index, 1, true);
                this.operateButtonNameArray.splice(index, 1, "解冻");
                this.$message({
                  type: "success",
                  message: "冻结成功!"
                });
              })
              .catch(() => {
                this.$message({
                  type: "info",
                  message: "已取消该操作！"
                });
              });
          })
          .catch(() => {});
      } else {
        this.$confirm("此操作解冻该员工，是否继续?", "提示", {
          confirmButtonText: "确定",
          cancelButtonText: "取消",
          type: "warning"
        })
          .then(() => {
            row.role = employeeRole[row.role]
            row.status = employeeStatus.WORKING
            this.$store
              .dispatch(UPDATE_EMPLOYEE, row)
              .then(response => {
                this.isModifyButtonDisabledArray.splice(index, 1, false);
                this.operateButtonNameArray.splice(index, 1, "冻结");
                this.$message({ type: "success", message: "解冻成功!" });
              })
              .catch(() => {
                this.$message({
                  type: "info",
                  message: "已取消该操作！"
                });
              });
          })
          .catch(() => {});
      }
    },  
    handleEdit(index, row) {
      if (this.editButtonName[index] === TABLE_BUTTON_TYPE[0]) {
        this.editTextByName = row.name;
        this.editTextByMail = row.mail;
        this.editTextByTelephone = row.telephone;
        // this.editTextByRole = row.role;
        this.editTextByRole = employeeRole[row.role];
        this.editTextByStatus = row.status;
        this.isEdited[index] = true;
        this.editButtonName.splice(index, 1, TABLE_BUTTON_TYPE[1]);
      } else {
        const parkingLot = {
          id: row.id,
          name: this.editTextByName,
          capacity: this.editTextByCapacity
        };
        this.$store.dispatch(UPDATE_EMPLOYEE, parkingLot);
        this.isEdited[index] = false;
        this.editButtonName.splice(index, 1, TABLE_BUTTON_TYPE[0]);
      }
    }
  }
};
</script>

<style scoped>
</style>
