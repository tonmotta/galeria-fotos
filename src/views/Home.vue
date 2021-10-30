<template>
  <ion-page>
    <ion-header>
      <ion-toolbar>
        <ion-title>Galeria de Fotos</ion-title>
      </ion-toolbar>
    </ion-header>
      <ion-content :fullscreen="true">
          <ion-grid>
    <ion-row>
      <ion-col size="6" :key="photo" v-for="photo in photos">
        <ion-img :src="photo.webviewPath"></ion-img>
      </ion-col>
    </ion-row>
  </ion-grid>
  <ion-fab vertical="bottom" horizontal="center" slot="fixed">
    <ion-fab-button @click="takePhoto()">
      <ion-icon :icon="camera"></ion-icon>
    </ion-fab-button>
  </ion-fab>
</ion-content>
  </ion-page>
</template>

<script lang="ts">
import { camera, trash, close } from 'ionicons/icons';
import { IonPage, IonHeader, IonFab, IonFabButton, IonIcon,
         IonToolbar, IonTitle, IonContent, IonGrid, IonRow, IonCol, IonImg } from '@ionic/vue';
import { ref, onMounted, watch } from 'vue';
import { Camera, CameraResultType, CameraSource, Photo } from '@capacitor/camera';
import { Filesystem, Directory } from '@capacitor/filesystem'
import { Storage } from '@capacitor/storage'

  const PHOTO_STORAGE = "photos";

  const photos = ref<UserPhoto[]>([]);

  interface UserPhoto {
  filepath: string;
  webviewPath?: string;
}

function usePhotoGallery() {

    const takePhoto = async () => {
      const cameraPhoto = await Camera.getPhoto({
        resultType: CameraResultType.Uri,
        source: CameraSource.Camera,
        quality: 100
      });
        const fileName = new Date().getTime() + '.jpeg';
        const savedFileImage = {
        filepath: fileName,
        webviewPath: cameraPhoto.webPath
};
photos.value = [savedFileImage, ...photos.value];
    };
    return {
        photos,
        takePhoto
    }
  }

  const cachePhotos = () => {
  Storage.set({
    key: PHOTO_STORAGE,
    value: JSON.stringify(photos.value)
  });
}

watch(photos, cachePhotos);

const loadSaved = async () => {
  const photoList = await Storage.get({ key: PHOTO_STORAGE });
  const photosInStorage = photoList.value ? JSON.parse(photoList.value) : [];

  for (const photo of photosInStorage) {
    const file = await Filesystem.readFile({
        path: photo.filepath,
        directory: Directory.Data
    });
    photo.webviewPath = `data:image/jpeg;base64,${file.data}`;
  }

  photos.value = photosInStorage;
}

onMounted(loadSaved);

  export default  {
  name: 'home',
  components: { IonPage, IonHeader, IonFab, IonFabButton, IonIcon,
         IonToolbar, IonTitle, IonContent, IonGrid, IonRow, IonCol, IonImg },
  setup() {
    const { takePhoto } = usePhotoGallery();

    return {
  photos, takePhoto, camera, trash, close
    }
  }
}

</script>

<style scoped>
#container {
  text-align: center;
  
  position: absolute;
  left: 0;
  right: 0;
  top: 50%;
  transform: translateY(-50%);
}

#container strong {
  font-size: 20px;
  line-height: 26px;
}

#container p {
  font-size: 16px;
  line-height: 22px;
  
  color: #8c8c8c;
  
  margin: 0;
}

#container a {
  text-decoration: none;
}
</style>