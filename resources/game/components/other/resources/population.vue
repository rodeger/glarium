<template>
    <div class="d-inline-block" title="Ciudadanos" v-if="cant>0||visible||disabled">
        <img :src="require('Img/icon/icon_citizen.png')" :class="disabled&&cant==0 ? 'opacity-5' : ''"> {{$money(cant)}}
    </div>
</template>

<script>
export default {
    name:'population',
    props:{
        cant:Number,
        visible:{
            type:Boolean,
            default:false
        },
        disabled:{
            type:Boolean,
            default:false
        }
    }
}
</script>
