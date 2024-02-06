<!-- HelloWorld.vue -->
<template>
  <div class="hello">
    <!-- Use the TableComponent and pass items as a prop -->
    <TableComponent :items="response || []" @add-row="handleAddRow" @dispatch="dispatcherData"
      @delete-row="handleDeleteRow" @edit-row="handleEditRow" />
  </div>
</template>


<script>
import axios from 'axios';
import TableComponent from './TableComponent.vue';


// 직렬화 함수 정의
function serializeArray(arr, paramName) {
  return arr.map(value => `${paramName}=${encodeURIComponent(value)}`).join('&');
}

export default {
  name: 'HelloWorld',

  components: {
    TableComponent,
  },

  mounted() {
    if (!this.response) {
      // 데이터를 이미 불러왔다면 다시 불러오지 않음
      this.dispatcherData('/api/list', '');
    }
  },

  data() {
    return {
      response: null,
    };
  },

  methods: {

    dispatcherData(url, data) {
      return this.httpMethod(url, data)
        .then(response => {
          this.response = response.data;
          if (url !== '/api/list') {
            this.dispatcherData('/api/list', '');
          }
        })
        .catch(error => {
          console.error('에러 발생', error);
        });
    },

    httpMethod(url, data) {
      switch (url) {
        case '/api/list':
          return axios.get(url);

        case '/api/saveChanges':
          console.log(data);
          return axios.post(url, data, this.header);

        case '/api/delete': {
          if (data.length > 0) {
            const queryString = serializeArray(data, 'list');

            // 쿼리스트링을 직접 URL에 추가
            const fullUrl = `${url}?${queryString}`;

            return axios.delete(fullUrl, this.header());
          } else {
            return Promise.resolve();
          }
        }
      }
    },

    header() {
      return {
        headers: {
          'Content-Type': 'application/json',
        },
      };
    },

    handleAddRow(newRow) {
      this.response.push(newRow);
    },

    // 매개변수는 배열
    handleDeleteRow(selectedDataId) {
      console.log("선택된 데이터의 시퀀스: ", selectedDataId);

      // this.response라는 배열에서 id속성의 값이 selectedDataId와 동일한 배열을 제외 시키자
      // selectedDataId 배열에 속하지 않은 항목들만 유지
      this.response = this.response.filter(item => !selectedDataId.includes(item.id));
    },

    handleEditRow(index) {
      this.response[index].editMode = true;
    },
  },
};
</script>

<style scoped>
h3 {
  margin: 40px 0 0;
}

ul {
  list-style-type: none;
  padding: 0;
}

li {
  display: inline-block;
  margin: 0 10px;
}

a {
  color: #42b983;
}
</style>
