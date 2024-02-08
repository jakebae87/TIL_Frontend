<!-- TableComponent.vue -->
<template>
  <div>
    <table class="table table-striped">
      <thead>
        <tr>
          <th>
            <input type="checkbox" v-model="selectAll" @change="toggleSelectAll" />
          </th>
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
import axios from 'axios';

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
    // 데이터 불러오기 및 검색
    getData(search) {
      axios
        .get('/api/list', {
          params: {
            searchValue: search
          }
        })
        .then(response => {
          this.$emit('update_list', response.data);
        })
        .catch(error => {
          console.error('에러 발생', error);
        });
    },

    // 저장 및 수정
    setData() {
      const selectedRows = this.items.filter(item => item.selected); // 체크박스 선택된 행 정보만 새로운 배열에 담는다.
      if (selectedRows.length == 0) {
        alert("수정된 행 또는 새로운 행이 선택되지 않았습니다.");
        return;
      }
      axios
        .post('/api/insertAndUpdate', selectedRows)
        .then(response => {
          alert(response.data);
          this.getData("");
        })
        .catch(error => {
          console.error('에러 발생', error);
        });
    },

    // 행 추가
    addRow() {
      const newRow = { id: '', user_nm: '', pwd: '', isNew: true };
      const lastId = this.items.length > 0 ? this.items[this.items.length - 1].id : 0;
      newRow.id = parseInt(lastId) + 1;

      this.$emit('push_row', newRow);
    },

    // 특정 데이터 행 수정폼으로 변환
    editRow(index) {
      this.$emit('edit_row', index);
    },

    // 공통 삭제버튼 구현
    deleteData() {
      const selectedRows = this.items.filter(item => item.selected);
      console.log("selectedRows" , selectedRows);

      if (selectedRows.length == 0) {
        alert("삭제할 행이 선택되지 않았습니다.");
        return;
      }

      if (window.confirm("선택된 행을 삭제하시겠습니까?")) {
        const selectedRowIds = selectedRows.map(row => row.id);
        this.$emit('delete_row', selectedRowIds);

        // 부모 컴포넌트에 배열 전달
        // this.$emit('dispatch', '/api/delete', selectedRows);
        axios
        .delete('/api/delete',{
          data: selectedRows,
        })
        .then(response => {
          alert(response.data);
        })
        .catch(error => {
          console.error('에러 발생', error);
        });
      }
    },

    toggleSelectAll() {
      // 전체 선택 체크박스의 상태에 따라 각 행의 선택 상태를 변경
      this.items.forEach(item => {
        item.selected = this.selectAll;
      });
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