<template>
  <div class="p-8 bg-gray-900 min-h-screen space-y-6 text-white">

    <header class="text-center py-4">
      <h1 class="text-3xl font-bold text-white tracking-wide border-b-2 border-blue-500 inline-block pb-1">
        ONLINE API HANDLER
      </h1>
      <p class="text-sm text-gray-400 mt-1">Build, send, and view custom HTTP requests</p>
    </header>


    <section class="bg-gray-800 rounded-xl shadow p-6 space-y-4">
      <div class="grid grid-cols-12 gap-4">
        <input
          @click="set_state('Url')"
          v-model="url"
          type="text"
          placeholder="http://your_url"
          class="col-span-8 px-4 py-2 rounded-md bg-gray-700 text-white focus:outline-blue-400 border border-gray-600"
        />

        <select v-model="req_method" class="col-span-2 px-2 py-2 rounded-md bg-gray-700 text-white border border-gray-600">
          <option v-for="meth in req_ava" :key="meth" :value="meth">{{ meth.toUpperCase() }}</option>
        </select>

        <button
          @click="sendRequest"
          class="col-span-2 bg-blue-700 text-white rounded-md hover:bg-blue-800 px-4 py-2 text-center"
        >
          Send
        </button>
      </div>

      <div class="flex flex-wrap gap-2">
        <button
          v-for="item in data"
          :key="item"
          @click="set_state(item)"
          :class="`${clas.option} px-4 py-2 rounded-md hover:bg-blue-900 transition`"
        >
          {{ item }}
        </button>

        <button
          type="button"
          @click="res_ch"
          class="px-4 py-2 rounded-md text-white"
          :class="res_ar()"
        >
          Response
        </button>
      </div>
    </section>


    <section class="bg-gray-800 rounded-xl shadow-md p-6">
      <div class="text-lg font-semibold border-b border-gray-700 pb-2 mb-4 text-white">
        {{ cur_state }}
      </div>

    
      <textarea
        v-if="!['Params', 'Header', 'Url', 'Response'].includes(cur_state)"
        v-model="temp"
        placeholder="Input Body"
        class="w-full h-40 border border-gray-600 p-3 rounded-md bg-gray-700 text-white"
      ></textarea>

 
      <div v-if="cur_state === 'Response'">
        <pre class="bg-gray-700 p-4 rounded text-sm overflow-auto text-white">{{ response }}</pre>
      </div>


      <div v-if="['Params', 'Header'].includes(cur_state)" class="space-y-4">
        <div class="grid grid-cols-12 gap-2 items-center">
          <span class="col-span-3 font-medium text-center">Key</span>
          <span class="col-span-3 font-medium text-center">Value</span>
        </div>

        <div
          v-for="(item, index) in this[cur_state]"
          :key="index"
          class="grid grid-cols-12 gap-2 items-center"
        >
          <span class="col-span-3 bg-gray-700 p-2 rounded text-center">{{ item[0] }}</span>
          <span class="col-span-3 bg-gray-700 p-2 rounded text-center">{{ item[1] }}</span>
          <button class="col-span-2 text-red-400" @click="del(index)">Delete</button>
        </div>

        <div class="grid grid-cols-12 gap-2 items-center">
          <input
            type="text"
            placeholder="Key"
            v-model="key"
            class="col-span-3 px-2 py-1 border border-gray-600 rounded bg-gray-700 text-white"
          />
          <input
            type="text"
            placeholder="Value"
            v-model="value"
            class="col-span-3 px-2 py-1 border border-gray-600 rounded bg-gray-700 text-white"
          />
          <button
            @click="add_l"
            class="col-span-2 bg-green-600 text-white px-2 py-1 rounded hover:bg-green-700"
          >
            Add
          </button>
        </div>

        <div v-if="cur_state === 'Params' && params_out" class="text-sm mt-4">
          <span class="font-semibold">Generated Params:</span>
          <div class="text-gray-400">{{ params_out }}</div>
        </div>
      </div>
    </section>

    <section class="bg-gray-800 rounded-xl shadow-md p-6">
      <h2 class="text-xl font-semibold border-b border-gray-700 pb-2 mb-4 text-white">Request History</h2>
      <div v-if="history.length === 0" class="text-gray-400">No history yet.</div>
      <ul v-else class="space-y-2">
        <li
          v-for="(entry, index) in history"
          :key="index"
          class="bg-gray-700 p-3 rounded-md text-sm"
        >
          <p><strong class="text-blue-400 cursor-pointer">{{ entry.method.toUpperCase() }}</strong> → <span class="text-gray-300">{{ entry.url }}</span></p>
          <p class="text-green-400">Status: {{ entry.response.status }} - {{ entry.response.statusText }}</p>
          <button
          class="rounded-sm px-3 mt-4 h-10 bg-blue-800 hover:bg-sky-700"
          @click="selectHistory(entry)" 
          
          >
            Show More
          </button>
          <div v-if="selectedHistory === entry" class="mt-2 bg-gray-800 p-2 rounded border border-gray-600">
            <p><strong class="text-white">Headers:</strong> <pre class="text-gray-300">{{ JSON.stringify(entry.response.headers, null, 2) }}</pre></p>
            <p><strong class="text-white">Data:</strong> <pre class="text-gray-300">{{ JSON.stringify(entry.response.data, null, 2) }}</pre></p>
            <p><strong class="text-white">Request Config:</strong> <pre class="text-gray-300">{{ JSON.stringify(entry.response.config, null, 2) }}</pre></p>
          </div>
        </li>
      </ul>
    </section>
  </div>
</template>

<script>
import axios from 'axios';
export default {
  data() {
    return {
      data: ['Header', 'Params', 'Body', 'Auth'],
      url: '',
      Body: '',
      Header: [[3, 4]],
      Params: [[1, 2]],
      Auth: null,
      cur_state: 'url',
      temp: null,
      key: null,
      value: null,
      clas: {
        option:
          'm-1 text-center rounded-sm bg-gray-700 text-white focus:outline-2 focus:outline-blue-300 hover:text-blue-300 hover:outline-blue-300',
      },
      req_method: 'get',
      req_ava: ['get', 'post', 'put', 'patch', 'delete', 'head', 'options'],
      response: null,
      received: false,
      history: [],
      selectedHistory: null
    };
  },
  computed: {
    params_out() {
      let params_string = '?';
      if (this.Params.length > 0) {
        let m = this.Params.length;
        for (let i = 0; i < m; i++) {
          params_string += this.Params[i][0].toString() + '=' + this.Params[i][1].toString();
          if (i == m - 1) break;
          params_string += '&';
        }
        return params_string;
      } else {
        return null;
      }
    },
    params_req() {
      if (this.Params.length <= 0) return null;
      let param_obj = {};
      for (let i = 0; i < this.Params.length; i++) {
        param_obj[this.Params[i][0].toString()] = this.Params[i][1].toString();
      }
      return param_obj;
    },
    header_req() {
      if (this.Header.length <= 0 && this.Auth) return null;
      let header_obj = {};
      if (this.Auth) {
        header_obj['Authorization'] = this.Auth.toString();
      }
      for (let i = 0; i < this.Header.length; i++) {
        header_obj[this.Header[i][0].toString()] = this.Header[i][1].toString();
      }
      return header_obj;
    },
  },
  watch: {
    temp(newVal) {
      if (this.cur_state && this.$data.hasOwnProperty(this.cur_state)) {
        this[this.cur_state] = newVal;
      }
    },
    cur_state(newVal) {
      this.temp = this[this.cur_state];
    },
  },
  methods: {
    res_ar() {
      return this.received ? 'bg-green-600' : 'bg-gray-600';
    },
    res_ch() {
      this.cur_state = 'Response';
    },
    set_state(val) {
      this.cur_state = val;
    },
    add_l() {
      if (this.key != null && this.value != null) {
        this[this.cur_state].push([this.key, this.value]);
        this.key = null;
        this.value = null;
      }
    },
    del(ind) {
      this[this.cur_state].splice(ind, 1);
    },
    async sendRequest() {
      this.received = false;
      this.response = null;
      try {
        const res = await axios.request({
          method: this.req_method.toString(),
          url: this.url.toString() + (this.params_out || ''),
          headers: this.header_req,
          params: this.params_req,
          data: this.Body ? JSON.parse(this.Body) : undefined,
        });
        const formatted = this.recordAndFormatResponse(res);
        this.response = formatted;
        this.received = true;
        this.history.unshift({
          method: this.req_method,
          url: this.url,
          response: formatted,
        });
      } catch (err) {
        console.error('API Error:', err);
      }
    },
    recordAndFormatResponse(re) {
      return {
        status: re.status,
        statusText: re.statusText,
        headers: re.headers,
        data: re.data,
        config: {
          method: re.config?.method,
          url: re.config?.url,
          headers: re.config?.headers,
          params: re.config?.params,
          dataSent: re.config?.data,
        },
        timestamp: new Date().toISOString(),
      };
    },
    selectHistory(entry) {
      this.selectedHistory = this.selectedHistory === entry ? null : entry;
    }
  },
};
</script>

<style scoped>
textarea:focus {
  outline: 2px solid #3b82f6;
}
</style>
