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
            <img v-if="avatar" :src="avatar" />
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
        .post("/sdapi/v1/txt2img", {
          prompt:
            `((${this.gender})), ((${this.hairColor} hair)), ((${this.eyeColor} eyes)), ((${this.hairLength} hair)), ${beardTextPos}` +
            `young, face portrait, ((realistic)), ((full face)), ((face centered)), elegant, ((plain color background))`,
          negative_prompt:
            `${beardTextNeg}${genderTextNeg}, out of frame, border, frame, multiple background, text, words, writing, unnatural, poorly drawn face, ugly, ` +
            `disfigured, deformed, watermark, signature, cut off, low contrast, underexposed, overexposed, bad art, beginner, amateur, distorted face`,
          steps: 20,
          sampler_index: "Heun",
          cfg_scale: 7,
          width: 512,
          height: 512,
          restore_faces: false,
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
