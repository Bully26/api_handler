<template>
    <div class="flex-row m-10 bg-blue-50 h-110 px-2">
     <div class="grid grid-cols-20 gap-2 bg-white rounded-lg my-2">
         
            <input  
            @click="set_state('Url')"
            tabindex="0"
            type="text" placeholder="http://your_url" 
            class='w-full m-1 bg-green-100 col-span-2  rounded-sm bg-slate-100 text-stone-500 focus:outline-2 focus:outline-blue-300 hover:outline-blue-300 m-1 col-span-4' 
            v-model="url">
         <div v-for="item in data" class="col-span-2">
            <div
            @click="set_state(item)"
            tabindex="0"
            :class="clas.option"
            >
            {{ item }}
            </div>
         </div>
        
         <button type="button"  class="col-span-2 bg-green-200 rounded-lg"
         :class="res_ar()" 
         :disabled="false"
         @click="res_ch"
         >
         Response
        </button>
        <div class="flex">
             <select v-model="req_method" >
                <option v-for="meth in req_ava" :value="meth">{{meth}}</option>
             </select>
              <div  @click='sendbruh' class="my-1 text-center bg-blue-400 text-white rounded-xs col-span-2 w-full">
               send
            </div>
        </div>
     
     </div>
     <div class="bg-white h-80 outline-1 rounded-sm">
          <div  class="px-5 text-sm bg-slate-100 border-b-3 border-slate-500" >
            {{cur_state}}
          </div>
          <textarea v-if="cur_state!='Params' && cur_state!='Header' && cur_state!='Url' && cur_state!='Response'" type="text"  class="border-solid border-2 rounded-lg w-140 h-60 m-3 p-2" placeholder="input" v-model="temp"></textarea>
          <div v-if="cur_state=='Response'">
             {{response}}
          </div>
          <div v-if="cur_state=='Params' || cur_state=='Header'" class="grid grid-cols-12 grid-rows-10">
              

               <div label='box' class="col-start-4 col-end-12 mt-3">
                <div class="flex mb-2">
                    
                     <div class="w-35 text-center bg-slate-200">key </div>
                     <div class="w-2"></div>
                     <div class="w-35 text-center bg-slate-200">value</div>


                      
                </div>
                <div>
                         <div class="flex" v-if="this[this.cur_state].length > 0" v-for="(item,index) in this[this.cur_state]" :key="index">
                                <div class='w-35 text-center bg-slate-100 border-b-2 border-slate-300' >{{item[0]}}</div>
                                <div class="w-2 "></div>
                                <div class='w-35 text-center bg-slate-100 border-b-2 border-slate-300'>{{item[1]}}</div>
                                <div class="w-5"></div>
                                <button class='w-15 text-center ' @click="del(index)">delete</button>
                        
                        </div>
                        
                        <div class="flex ">
                            <input type="text text-center " class="w-35 bg-white outline" placeholder="key" v-model="key">
                            <div class="w-2"></div>
                            <input type="text text-center " class="w-35 bg-white outline" placeholder="value" v-model="value">
                            <div class="w-5"></div>
                              <div @click="add_l" class="w-15 text-center bg-red-500">
                               add
                            </div>
                        </div>
                        <div v-if="cur_state=='Params' && params_out!=null" class="mt-5">
                            <div>
                                Params:
                                <div class="text-slate-400 border-b-3 w-92">
                                    {{params_out}}
                                </div>
                            </div>

                        </div>        
                      
                </div>
               </div>
               
          </div>

     </div>
    </div>
</template>

<script>
import axios from 'axios';
export default {

    data(){
        return {
            data:['Header','Params','Body','Auth'],
            url: '',  
            Body: '', 
            Header:[[3,4]],
            Params:[[1,2]], 
            Auth:null,
            cur_state:'url',
            temp:null,
            key:null,
            value:null,
            clas:{
                option :"m-1 text-center rounded-sm bg-slate-100 text-stone-500 focus:outline-2  focus:outline-blue-300 focus:text-blue-300 hover:text-blue-300 hover:outline-blue-300"
            },
            req_method:'get',
            req_ava:['get', 'post', 'put', 'patch', 'delete', 'head', 'options'],
            response:null,
            received:false
        }
    },
    computed:{

        params_out(){
            let params_string='?';
            if(this.Params.length>0)
             {
                let m=this.Params.length;
                for (let i = 0; i < m; i++){
                    params_string+= this.Params[i][0].toString() +'='+ this.Params[i][1].toString();
                    if(i==m-1)break;
                    params_string+='&';
                }
                return params_string;
             }else{
                return null;
             }
            
        },
        params_req(){
             if(this.Params.length<=0)return null;

            let param_obj={};
            for(let i=0;i<this.Params.length;i++)
            {
                  param_obj[this.Params[i][0].toString()]=this.Params[i][1].toString();
            }
            return param_obj
        }
        ,
        header_req(){
             
            
            if(this.Header.length<=0 && this.Auth)return null;
            let header_obj={};

            if (this.Auth) {
                    header_obj['Authorization'] = this.Auth.toString();
                    }

            for(let i=0;i<this.Header.length;i++)
            {
                   header_obj[this.Header[i][0].toString()]=this.Header[i][1].toString();
            }
            return header_obj;
        }
    },
    watch:{
            temp(newVal) {
            if (this.cur_state && this.$data.hasOwnProperty(this.cur_state)) {
             this[this.cur_state] = newVal;
            }
            },
            cur_state(newVal){
             this.temp=this[this.cur_state];
            }
    }
    ,
    methods:{
        res_ar(){
            return this.received?'bg-green-200':'bg-slate-500';
        },
        res_ch(){
            this.cur_state='Response';
        },
        set_state(val){
            this.cur_state=val
        },
        assign_asso(val){
            if(val=='url')return this.url;

        },
        add_l(){
            if(this.key!=null && this.value!=null)
            {
                this[this.cur_state].push([this.key,this.value]);
                this.key=null;
                this.value=null;
            }
        },
        del(ind)
        {
            this[this.cur_state].splice(ind,1);
        },
        sendbruh(){
            this.received=false;
            this.response=null;
            this.fetchCustomApi();
        }
        ,
        async fetchCustomApi() {
            try {
                this.response = await axios.request({
                method: this.req_method.toString(),
                url: this.url.toString()+this.params_out.toString(), 
                headers: this.header_req,
                params: this.params_req,
                data:this.Body?JSON.parse(this.Body) : undefined
                })
                this.response=this.recordAndFormatResponse(this.response);
                this.received=true;

                console.log('Response:', response)
            } catch (err) {
                console.error('API Error:', err)
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
                    dataSent: re.config?.data
                },
                timestamp: new Date().toISOString()
            };
        }

    }
    

}

</script>
