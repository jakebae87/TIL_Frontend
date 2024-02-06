<!-- TableComponent.vue -->
<template>
  <div>
    <div class="common-buttons">
      <button @click="getData">조회</button>
      <button @click="saveChanges">저장</button>
      <button @click="addRow">추가</button>
      <button @click="deleteData">삭제</button>
    </div>

    <table class="table table-striped">
      <thead>
        <tr>
          <th>
            <input type="checkbox" v-model="selectAll" @change="toggleSelectAll" />
          </th>
          <!-- <th>번호</th> -->
          <th>아이디</th>
          <th>비밀번호</th>
          <th>작업</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="(item, index) in items" :key="index">
          <td>
            <input type="checkbox" v-model="item.selected" />
          </td>
          <!-- <td>
            {{ item.id }}
          </td> -->
          <td>
            <span v-if="item.isNew || item.editMode">
              <input v-model="item.user_nm" type="text" />
            </span>
            <span v-else>
              {{ item.user_nm }}
            </span>
          </td>
          <td>
            <span v-if="item.isNew || item.editMode">
              <input v-model="item.pwd" type="password" />
            </span>
            <span v-else>
              {{ item.pwd }}
            </span>
          </td>
          <td>
            <button @click="editRow(index)">수정</button>
          </td>
        </tr>
      </tbody>
    </table>
  </div>
</template>


<script>
export default {
  props: {
    items: {
      type: Array,
      default: () => [],
    },
  },

  data() {
    return {
      lastId: 0,
      selectAll: false,
    };
  },

  methods: {

    getData() {
      alert("목록 조회 완료");
      this.$emit('dispatch', '/api/list');
    },

    // 행 추가
    addRow() {
      const newRow = { id: '', user_nm: '', pwd: '', isNew: true };
      // DB에 저장된 마지막 행의 id 값에 1을 추가
      const lastId = this.items.length > 0 ? this.items[this.items.length - 1].id : 0;
      newRow.id = parseInt(lastId) + 1;

      this.$emit('add-row', newRow);
    },


    // 특정 데이터 행 수정폼으로 변환
    editRow(index) {
      this.$emit('edit-row', index);
    },


    saveChanges() {
      const updatedRows = this.items.filter(item => item.editMode && item.selected);
      const newRows = this.items.filter(item => item.isNew && item.selected);

      if (updatedRows.length === 0 && newRows.length === 0) {
        alert("수정된 행 또는 새로운 행이 선택되지 않았습니다.");
        return;
      }

      // 서버로 전송할 데이터 구성
      const requestData = {
        updates: updatedRows.map(row => ({ id: row.id, user_nm: row.user_nm, pwd: row.pwd })),
        inserts: newRows.map(row => ({ id: row.id, user_nm: row.user_nm, pwd: row.pwd })),
      };

      // 서버에 데이터 전송
      this.$emit('dispatch', '/api/saveChanges', requestData); // ArrayList<Map>으로 서버에 전송한다.
    },


    // 공통 삭제버튼 구현
    deleteData() {
      const selectedRows = this.items.filter(item => item.selected); // 체크박스 선택된 행 정보만 새로운 배열에 담는다.

      const selectedExistingRows = this.items.filter(item => item.selected && !item.isNew); // 기존 데이터에서 선택된 행 정보만 새로운 배열에 담는다.
      // 로그 남겨놓기 (isNew)

      if (selectedRows.length === 0) {
        alert("선택된 행이 없습니다.");
        return;
      }

      // 기존 데이터 삭제
      if (window.confirm("선택된 행을 삭제하시겠습니까?")) {

        // 선택된 행들을 화면에서 제거
        const selectedRowIds = selectedRows.map(row => row.id);
        console.log("TableComponent_selectedRowIds: " + selectedRowIds);
        this.$emit('delete-row', selectedRowIds);

        // 선택된 행들의 ID 배열 생성
        const selectedExistingRowsIds = selectedExistingRows.map(row => row.id); //배열로 선택된 후 삭제버튼 클릭시 해당 id 정보가 담긴다.

        // 부모 컴포넌트에 배열 전달
        this.$emit('dispatch', '/api/delete', selectedExistingRowsIds);
      }
    },

    toggleSelectAll() {
      // 전체 선택 체크박스의 상태에 따라 각 행의 선택 상태를 변경
      this.items.forEach(item => {
        item.selected = this.selectAll;
      });
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