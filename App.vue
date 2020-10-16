<template>
  <view class="container">
    <cameraModule />
    <loadingScreen />
    <carPage></carPage>
  </view>
</template>

<script>
import axios from 'axios';
import * as Permissions from 'expo-permissions';
import { Camera } from 'expo-camera';
import * as ImageManipulator from "expo-image-manipulator";
import './package-loader';
import EventBus from './eventBus';

import cameraModule from './components/cameraModule'
import carPage from './components/carPage';
import loadingScreen from './components/loadingScreen';

export default {
  data() {
    return {
    hasCameraPermission: false,
    type: Camera.Constants.Type.back,
    pic: null,
    loading: false,
    carLoaded: false,
    carData: null,
    fuelData: null,
    loadingText: ''
    }
  },
  mounted() {
  },
  components: { carPage, cameraModule, loadingScreen },
  methods: {
  }
}
</script>

<style>
.container {
  flex: 1;
}

.text-color-primary {
  color: blue;
}

.shutterBtn {
  color: black;
}
</style>
