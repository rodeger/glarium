<template>
    <div class="box">
        <div class="gtitle text-center">Area: {{$t('researchCategories['+category+']')}}</div>
        <div class="d-flex my-3 py-1 px-3 fondo">
            <div class="flex-1 d-flex">
                <div class="mr-2"><img :src="require('Img/icon/icon_scientist.png')"></div>
                <div>{{$t('research2.scientist')}}: {{total_scientists}}</div>
            </div>
            <div class="flex-1 d-flex justify-content-center">
                <div class="mr-2"><img :src="require('Img/icon/icon_pi.png')"></div>
                <div>{{$t('research2.pi')}} {{$money(research_point)}}</div>
            </div>
            <div class="flex-1 d-flex justify-content-end">
                <div class="mr-2"><img :src="require('Img/icon/icon_research_time.png')"></div>
                <div>{{$t('other.perHour')}}: {{research_point_hour}}</div>
            </div>
        </div>
        <div class="d-flex">
            <div class="flex-1 mr-2 lista">
                <div v-for='(research,index) in researchs' :key='index' @click='changeResearch(research,index)' class="d-flex py-1 px-2 research" :class="getClass(research)">
                    <div class="mr-2">{{research.level}}.</div>
                    <div class="research-name">{{$t('research['+research.id+'].name')}}</div>
                </div>
            </div>
            <div class="flex-2 p-2 borde">
                <div class="gtitle mb-2">{{$t('research['+selected.id+'].name')}}</div>
                <div class="text text-justify">{{$t('research['+selected.id+'].text')}}</div>
                <div class="d-flex mt-3">
                    <div class="costos mr-2">{{$t('other.cost')}}: </div>
                    <div>{{$money(selected.cost)}} <img :src="require('Img/icon/icon_pi.png')"></div>
                </div>
                <div class="text-right mt-3">
                    <div v-if='selected.level>maxLevel+1' class="text-danger">{{$t('research2.previusLevel')}}</div>
                    <div v-else-if='researchs[selected.index].finish'>{{$t('research2.finished')}}</div>
                    <div v-else>
                        <div class="btnGeneral" v-if="research_point>=selected.cost" @click='investigar'>{{$t('research2.button')}}</div>
                        <div class="text-danger" v-else>{{$t('research2.dontHave')}}</div>
                    </div>
                </div>
            </div>
        </div>
    </div>
</template>

<script>
import axios from 'axios'
import $notification from 'Stores/notification'
import $resources from 'Stores/resources'
import $config from 'Stores/config'

export default {
    name: 'Investigaciones',
    props:['data_aux','category'],
    data(){
        return {
            data:{},
            researchs:[],
            selected:{},
            maxLevel:0
        }
    },
    methods:{
        changeResearch(research,index){
            this.selected = {
                id:research.id,
                level:research.level,
                cost:research.cost,
                index:index
            };
        },
        getClass(research){
            var clase = '';
            clase += research.id==this.selected.id ? 'active ' : '';
            clase += research.level>this.maxLevel+1 ? 'inactive ' : '';
            return clase;
        },
        investigar(){
            axios.post('research/'+this.selected.id)
            .then(res =>{
                if(res.data=='ok'){
                    $resources.commit('research',{research_point:this.selected.cost})
                    this.data.user_research.push(this.selected.id)
                    this.initData()
                    $notification.commit('show',{advisor:3,type:true})
                }else{
                    $notification.commit('show',{advisor:3,type:false,message:res.data});
                }
            })
            .catch(err =>{
                $notification.commit('show',{advisor:3,type:false,message:err});
            })
        },
        initData(){
            this.researchs = this.data.research.filter(x =>{
                return x.category_id == this.category;
            });
            var userResearch = this.researchs.filter(x =>{
                var res =  this.data.user_research.includes(x.id);
                x.finish = res;
                return res;
            });
            if(userResearch.length>0){
                this.maxLevel = Math.max.apply(Math, userResearch.map(function(o) { return o.level; }))
                if(userResearch.length==this.researchs.length){
                    this.maxLevel -= 1;
                }
            }else{
                this.maxLevel = 0
            }
            this.researchs.some((research,index) =>{
                if(research.level == this.maxLevel+1){
                    this.changeResearch(research,index)
                    return
                }
            });
        }
    },
    computed:{
        total_scientists(){
            return $resources.state.userResources.total_scientists;
        },
        research_point(){
            return $resources.state.userResources.research_point;
        },
        research_point_hour(){
            return this.$money_two(this.total_scientists);
        }
    },
    watch:{
        category(newval){
            this.initData()
        },
        data(){
            this.initData()
        }
    },
    beforeMount(){
        this.data = this.data_aux;
        this.initData()
    }
}
</script>

<style lang="scss" scoped>
 .box{
    font-size: 0.83rem;
 }
 .active,.fondo{
    background: #dec493;
 }
 .research{
     cursor: pointer;
 }
 .research:hover{
    background: #dec493;
    text-decoration: underline;
 }
 .text{
    line-height: 0.83rem;
 }
 .borde{
    border: 1px solid #e3c998;
 }
 .gtitle{
    font-size: 0.90rem;
 }
 .inactive{
    color: gray
 }
 .btnGeneral{
    display: inline-block;
    padding: 4px 25px;
 }
 .lista{
    overflow: hidden;
    user-select: none;
 }
 .research-name{
    white-space: nowrap;
 }
 .costos{
    color: #542c0f;
    font-weight: bold;
 }
</style>
