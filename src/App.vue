<template>
  <div id="app">  	
	<MainBlock
	class="mainblock"
    	@getImageData="getImageData"
    @getImageDataUrl="getImageDataUrl"
    	:inputHeight='300' 
    	:inputWidth='300' 
	></MainBlock> 	
  	<img class="img" :src="src"/>
  </div>
</template>

<script>
import MainBlock from './components/mainBlock'
export default {
  name: 'App',
  components: {
    MainBlock,
  },
  data(){
  	return{
  		src:""
  	}
  },
  methods:{
  	getImageData(imageData){
  		console.log(imageData)
  	},
  	getImageDataUrl(dataUrl){
  		this.src = dataUrl
  	}
  },
}
</script>

<style>
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
  display: flex;
  align-items: center;
  margin-left: 20%;
}

.mainblock{
	border: 1px solid gainsboro;
	border-radius: 2px;
}

.img{
	margin-top: 10px;
	margin-left: 20px;
}
</style>
