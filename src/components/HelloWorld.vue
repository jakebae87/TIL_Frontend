<template>
  <div class="hello">
    <div class="common-buttons">
      <input v-model ="searchValue" @keyup.enter="getData"/>
      <button @click="getData">조회</button>
      <button @click="setData">저장</button>
      <button @click="addRow">추가</button>
      <button @click="deleteData">삭제</button>
    </div>
    <TableComponent :items="response" ref="tableRef" 
    @push_row="pushRow" @update_list="updateList" @edit_row="editRow" @delete_row="deleteRow" />
  </div>
</template>

<script>
import TableComponent from './TableComponent.vue';

export default {
  name: 'HelloWorld',

  components: {
    TableComponent,
  },

  mounted() {
    this.getData();
  },

  data() {
    return {
      response: [],
      searchValue: '',
    };
  },

  methods: {

    //데이터 조회
    getData() {
      // $refs를 사용하여 자식 컴포넌트의 함수 호출
      this.response = this.$refs.tableRef.getData(this.searchValue);
    },

    //데이터 조회 후 리스트 업데이트
    updateList(datas) {
      this.response = datas;
    },

    //데이터 저장 및 수정
    setData() {
      this.$refs.tableRef.setData();
    },

    //새로운 행 추가
    addRow() {
      this.$refs.tableRef.addRow();
    },

    //새로운 행 추가 후 리스트 업데이트
    pushRow(newRow) {
      this.response.push(newRow);
    },

    //데이터 수정을 위해 input 태그 변경
    editRow(index) {
      this.response[index].editMode = true;
    },

    //데이터 및 새로운 행 삭제
    deleteData() {
      this.$refs.tableRef.deleteData();
    },

    //체크박스 선택된 행을 화면에서 제외
    deleteRow(row) {
      this.response = this.response.filter(item => !row.includes(item.id));
    }

  },
};
</script>

<style scoped>
/* 우측 상단 버튼 스타일 */
.common-buttons {
  text-align: right;
  margin-bottom: 2%;
  margin-right: 5%;
}

button {
  margin-right: 5px;
}
</style>
