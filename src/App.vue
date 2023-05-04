<template>
  <q-layout view="lHh Lpr lFf">
    <q-page-container>
      <q-card class="my-card">
        <q-card-section class="bg-primary text-white">
          <div class="text-h6">Avatar Generator</div>
          <div class="text-subtitle2">by Thomas Rüegg and Patrick Wissiak</div>
        </q-card-section>
        <q-linear-progress :value="progress" />

        <div class="q-my-md">
          <q-avatar class="avatar" size="120px" color="accent" square>
            <img v-if="avatar" :src="avatar" @click="downloadAvatar" />
          </q-avatar>
        </div>

        <q-separator />

        <div class="q-pa-lg">
          <q-radio v-model="gender" val="woman" label="Female" />
          <q-radio v-model="gender" val="man" label="Male" />

          <q-checkbox v-if="gender == 'man'" v-model="beard" label="Beard" />

          <q-select
            v-model="hairColor"
            :options="hairColors"
            label="Hair Color"
          />

          <q-select
            v-model="hairLength"
            :options="hairLengths"
            label="Hair Length"
          />
          <q-select v-model="eyeColor" :options="eyeColors" label="Eye Color" />
        </div>

        <q-card-actions align="right">
          <q-btn flat>Clear</q-btn>
          <q-btn
            flat
            color="primary"
            :disabled="
              loading || !hairColor || !hairLength || !gender || !eyeColor
            "
            @click="generateAvatar"
          >
            Generate
          </q-btn>
        </q-card-actions>
      </q-card>
    </q-page-container>
  </q-layout>
</template>

<script>
import { defineComponent } from "vue";

export default defineComponent({
  name: "App",
  data: () => ({
    avatar: undefined,
    loading: false,
    progress: 0,
    gender: undefined,
    hairColor: undefined,
    eyeColor: undefined,
    hairLength: undefined,
    beard: false,
    hairColors: [
      { value: "blonde", label: "Blonde" },
      { value: "brown", label: "Brown" },
      { value: "red", label: "Red" },
    ],
    hairLengths: [
      { value: "long", label: "Long" },
      { value: "short", label: "Short" },
    ],
    eyeColors: [
      { value: "blue", label: "Blue" },
      { value: "brown", label: "Brown" },
      { value: "green", label: "Green" },
    ],
  }),

  methods: {
    generateAvatar() {
      this.loading = true;
      const beardTextPos = this.beard ? "((beard)), " : "";
      const beardTextNeg = !this.beard ? "((beard)), " : "";
      const genderTextNeg = this.gender === "man" ? "woman" : "man";

      this.$api
        .post("/sdapi/v1/img2img", {
          init_images: [
            "data:image/jpeg;base64,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PC0AfZzEDg9+9Vm3Zx3rhPDnxR8IeKEUW10Ibg/8s245+prujgqCpDg9MHP8qAK7VE3WpD1qInvQBHUdPPSmUAf/1/2VGRz27VMobPJ47k8CuY8R+J9J8KWDX2qzBB/CnUtXyt4u+NHiDXGa20gmyteQMc7hQB9X6x4x8N+HkL6peKmOw+b+VeL+IP2hLOHdF4ftfNI6SZx+hr5WnnuLqQzXMrSOeTkmouAc4zQB3PiL4keLvExIvbxkiP8AAvHH1FcLgFtxyxPqc/zo+opfqKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKAFUsjh43ZCOm0kY/KvRvDvxV8Y+HNsUVyZ4B/Af8TXnP8AKgigD640D9oDRrzbFrlv9lc8b85/lXseleJNC12ISaXdpKD6kD9DX5xcH3Aq3Z3t7p0wuLGZonHoTj8qAP0pbIHvTeD92vkfwd8cNV0147LxIPtNv0EnTb+VfUukavp2vWKajpkolikGeOo/CgD/0PovxX4q1LxfqjahfyHb/DHnha5c9KXjGMZo/GgAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooADyB6V6B8PfHN54N1QPvLWEn+sQn06V5/wDhTX6H3oA//9H2SiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKQ9KWkPSgD//S9kooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACkPSlooA/9P2SiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKACiiigAooooAKKKKAP//Z",
          ],
          prompt:
            `((${this.gender})), ((${this.hairColor} ${this.hairLength} hair)), ((${this.eyeColor} eyes)), ${beardTextPos}` +
            `young, ((avatar style))`,
          negative_prompt:
            `${beardTextNeg}${genderTextNeg}, out of frame, border, frame, multiple background, text, words, writing, unnatural, poorly drawn face, ugly, ` +
            `disfigured, deformed, watermark, signature, cut off, low contrast, underexposed, overexposed, bad art, beginner, amateur, distorted face`,
          steps: 20,
          sampler_index: "Heun",
          cfg_scale: 7,
          denoising_strength: 0.9,
          width: 512,
          height: 512,
          restore_faces: true,
          tiling: false,
        })
        .then((res) => {
          this.loading = false;
          this.progress = 1;
          setTimeout(
            () => (this.avatar = "data:image/png;base64," + res.data.images[0]),
            1000 // Timeout to improve progress visualization
          );
        })
        .catch(() => (this.loading = false));
      this.showQueue();
    },
    async showQueue() {
      const res = await this.$api.get("/sdapi/v1/progress");
      this.progress = res.data.progress;
      setTimeout(async () => {
        if (this.loading) {
          this.showQueue();
        }
      }, 1000);
    },
    downloadAvatar() {
      const hiddenEl = document.createElement("a");
      hiddenEl.setAttribute(
        "href",
        document.getElementsByTagName("img")[0].src
      );
      hiddenEl.setAttribute("download", "generated-avatar.png");
      hiddenEl.style.display = "none";
      document.body.appendChild(hiddenEl);
      hiddenEl.click();
      document.body.removeChild(hiddenEl);
    },
  },
});
</script>

<style scoped>
.my-card {
  margin: 64px;
}

.avatar {
  margin-left: 50%;
  transform: translate(-50%);
}
</style>
