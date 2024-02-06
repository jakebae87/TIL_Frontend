<template>
  <div class="hello">
    <div class="common-buttons">
      <button @click="getData">조회</button>
      <button @click="setData">저장</button>
      <button @click="addRow">추가</button>
      <button @click="deleteData">삭제</button>
    </div>
    <TableComponent :items="response" ref="tableRef"
    @push_row="pushRow" @update_list="updateList"  />
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
    // response를 빈 배열로 초기화
    this.response = [];
    if (!this.response.length) {
      this.getData();
    }
  },

  data() {
    return {
      response: [],
    };
  },

  methods: {
    //데이터 조회
    getData() {
      // $refs를 사용하여 자식 컴포넌트의 함수 호출
      this.response = this.$refs.tableRef.getData();
    },
    //데이터 조회 후 리스트 업데이트
    updateList(datas) {
      this.response = datas;
    },

    //데이터 저장 및 수정
    setData() {
      this.$refs.tableRef.setData();
      // 메소드에서 받아온 데이터를 this.response에 넣어줘야 하지 않을까?
    },

    //새로운 행 추가
    addRow() {
      this.$refs.tableRef.addRow();
    },

    //새로운 행 추가 후 리스트 업데이트
    pushRow(newRow) {
      this.response.push(newRow);
    },

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
