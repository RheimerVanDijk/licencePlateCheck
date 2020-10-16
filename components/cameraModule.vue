<template>
	<view class="cameraModule">
		<view class="cameraView">
    	<camera class="camera" :type="this.type" pictureSize="300 x 100" ref="cameraModule" :flashMode="flashArray[flashState]" :zoom="parseFloat(zoomLevel)"/>
			<view class="cameraOverlay" />
			<nb-button iconLeft transparent class="flashClass" :on-press="changeFlash">
				<nb-icon class="flashAuto" v-if="flashState == 0" active name="flash-circle" type="MaterialCommunityIcons"/>
				<nb-icon class="flashOn" v-if="flashState == 1" active name="flash-circle" type="MaterialCommunityIcons"/>
				<nb-icon class="flashOn" v-if="flashState == 2" active name="flashlight" />
			</nb-button>
			<view class="zoomClass">
				<nb-button iconLeft transparent :on-press="zoomIn">
					<nb-icon class="zoomIcon" active name="pluscircleo" type="AntDesign"/>
				</nb-button>			
				<nb-button iconLeft transparent :on-press="zoomOut">
					<nb-icon class="zoomIcon" active name="minuscircleo" type="AntDesign"/>
				</nb-button>
			</view>

		</view>
		<view class="shutterBtnView">
      <nb-button iconLeft transparent primary :on-press="takePicture">
        <nb-icon class="zoomIcon" active name="camera" />
				<nb-text class="zoomIcon">Take photo</nb-text>
      </nb-button>
			<nb-button iconLeft transparent primary :on-press="openSearch">
				<nb-icon class="zoomIcon" active name="search"/>
				<nb-text class="zoomIcon">Search licence</nb-text>
			</nb-button>
    </view>
		<nb-container class="searchPopup" v-if="searchState">
			<nb-header>
			<nb-left>
				<nb-button transparent :on-press="closeSearch">
					<nb-text>Back</nb-text>
				</nb-button>
			</nb-left>
			<nb-body>
				<nb-title>Search</nb-title>
			</nb-body>
			<nb-right>
				<nb-button transparent>
				</nb-button>
			</nb-right>
		</nb-header>
			<nb-content class="searchContent">
        <nb-item floatingLabel>
          <nb-label >Licence plate</nb-label>
          <nb-input v-model="searchLicence" />
        </nb-item>
		<nb-button block class="searchBtn" :on-press="search">
        	<nb-text>Search</nb-text>
      	</nb-button>
			</nb-content>
		</nb-container>
	</view>
</template>

<script>
import axios from 'axios';
import * as Permissions from 'expo-permissions';
import { Camera } from 'expo-camera';
import * as ImageManipulator from "expo-image-manipulator";
import * as Haptics from 'expo-haptics';
import EventBus from '../eventBus';
import { Alert } from "react-native";

export default {
	components: {Camera},
	data() {
		return {
			hasCameraPermission: false,
			type: Camera.Constants.Type.back,
			searchLicence: '',
			searchState: false,
			flashState: 0,
			flashArray: ['auto','on','torch'],
			zoomLevel: 0
		}
	},
	mounted() {
		Permissions.askAsync(Permissions.CAMERA)
			.then(status => {
			hasCameraPermission = status.status == "granted" ? true : false;
			}).catch((err)=>{
				console.log(err);
			});
	},
	methods: {
		async takePicture() {
			Haptics.notificationAsync(Haptics.NotificationFeedbackType.Success)
			EventBus.$emit('changeLoadingState', true)
			EventBus.$emit('changeLoadingText', 'Taking picture...')
			await this.$refs.cameraModule.takePictureAsync()
				.then(data => {
					this.compressPicture(data)
				})
		},
		async compressPicture(data) {
      		await ImageManipulator.manipulateAsync(data.uri, [{resize: {width: 375, height: 500}},{crop: {originX: 50, originY: 200, width :275, height: 100}}], {compress: 0.5, base64: true })
        		.then(data => {
					this.readLicence(data.base64)
					EventBus.$emit('changeLoadingText', 'Reading licence plate...')
        		})
		},
		readLicence(base64) {
			let secret_key = "sk_0464e03e37f79c8093a536ac";
			let url = "https://api.openalpr.com/v2/recognize_bytes?recognize_vehicle=1&country=eu&secret_key=" + secret_key;
			let xhr = new XMLHttpRequest();
			xhr.open("POST", url);

			xhr.send(base64)
			xhr.onreadystatechange = () => {
				if (xhr.readyState == 4) {
				let response = JSON.parse(xhr.responseText)
				EventBus.$emit('changeLoadingText', 'Retreving licence plate information...')
				console.log(response.results, response.results.length)
				if (response.results.length > 0) {
					this.getLicence(response.results[0].candidates[0].plate, 0)
				} else {
					EventBus.$emit('changeLoadingState', false)
					Alert.alert("Couldn't find licence plate", "Take an other picture")
				}
			}
      }
		},
		getLicence(plate, type) {
			axios.get(`https://opendata.rdw.nl/resource/8ys7-d773.json?kenteken=${plate}`)
				.then(resp => {
					EventBus.$emit('fuelData', resp)
				})
			axios.get(`https://opendata.rdw.nl/resource/m9d7-ebf2.json?kenteken=${plate}`)
				.then(resp => {
					console.log(resp.data.length)
					if(resp.data.length > 0) {
						EventBus.$emit('changeLoadingState', false)
						EventBus.$emit('carData', resp)
						EventBus.$emit('carPageState', true)
					} else {
						EventBus.$emit('changeLoadingState', false)
						if(type == 0) {
							Alert.alert("Couldn't find licence plate", "Take an other picture")
						} else if (type == 1) {
							Alert.alert("Couldn't find licence plate", "Try again, make sure you've typed the licence plate correct")
						}
					}
	
				})
		},
		search() {
			Haptics.notificationAsync(Haptics.NotificationFeedbackType.Success)
			this.searchState = false
			EventBus.$emit('changeLoadingState', true)
			let toUpperCase = this.searchLicence.toUpperCase()
			let regexString = toUpperCase.replace(/-|\s/g,"")
			console.log(regexString)
			this.getLicence(regexString, 1)
		},
		openSearch() {
			Haptics.notificationAsync(Haptics.NotificationFeedbackType.Success)
			this.searchState = true
		},
		closeSearch() {
			Haptics.notificationAsync(Haptics.NotificationFeedbackType.Success)
			this.searchState = false
		},
		changeFlash() {
			Haptics.notificationAsync(Haptics.NotificationFeedbackType.Success)
			if (this.flashState != 2) {
				this.flashState++
			} else {
				this.flashState = 0
			}
		},
		zoomIn() {
			Haptics.notificationAsync(Haptics.NotificationFeedbackType.Success)
			if (this.zoomLevel < 0.7) {
				this.zoomLevel += 0.1
			}
		},
		zoomOut() {
			Haptics.notificationAsync(Haptics.NotificationFeedbackType.Success)
			if (this.zoomLevel > 0.00001) {
				this.zoomLevel -= 0.1
			}
		}
	}
}
</script>

<style>
.cameraModule {
	flex: 1;
}

.cameraView {
	position: relative;
}

.camera {
  width: 375px;
  height: 500px;
}

.cameraOverlay {
  width: 100%;
  height: 500px;
  border-top-width: 200px;
  border-top-color: rgba(0, 0, 0, 0.5);

  border-bottom-width: 200px;
  border-bottom-color: rgba(0, 0, 0, 0.5);

  border-left-width: 50px;
  border-left-color: rgba(0, 0, 0, 0.5);

  border-right-width: 50px;
  border-right-color: rgba(0, 0, 0, 0.5);
  position: absolute;
  top: 0;

}

.shutterBtnView {
  flex: 1;
  justify-content: space-around;
  align-items: center;
	flex-direction: row;
}

.searchPopup {
	position: absolute;
	top: 0;
	left: 0;
	height: 100%;
	width: 100%;
	background-color: white;
	flex: 1;
}

.searchContent {
	padding-top: 20px;
	padding-left: 30px;
	padding-right: 30px;
}

.searchBtn {
	margin-top: 20px;
}

.flashClass {
	position: absolute;
	bottom: 10px;
	left: 0;
}

.flashOn {
	color: #F4B200;
}

.zoomClass {
	position: absolute;
	bottom: 10px;
	right: 10px;
}

.flashAuto {
	color: white;
}

.zoomIcon {
	color: white;
}

.centerBtn {
	align-items: center;
	justify-content: center;
}
</style>