<template>
	<view class="loadingScreen" v-if="loadingScreenState">
		<text>{{ loadingText }}</text>
		<nb-spinner color="#000000"/>
	</view>
</template>

<script>
import EventBus from '../eventBus';

export default {
	data() {
		return {
			loadingText: '',
			loadingScreenState: false
		}
	},
	mounted() {
		EventBus.$on('changeLoadingText', (text) => {
      		this.loadingText = text;
		})
		
		EventBus.$on('changeLoadingState', (state) => {
      		this.loadingScreenState = state;
    	})
	}
}
</script>

<style>
.loadingScreen {
	width: 100%;
	height: 100%;
	position: absolute;
	top: 0;
	background-color: rgba(255, 255, 255, 0.7);
	justify-content: center;
	align-items: center;
}
</style>