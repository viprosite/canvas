<template>
  <div class="canvas-box">
    <canvas ref="canvas" width="500" height="500"></canvas>
    <!-- <div>
      <img v-for="item in imgList" :src="item" alt="" />
    </div> -->
  </div>
</template>

<script>
/**
 * 整个canvas向左移动 === 每个小图向左同时移动相同量
 */
import { imgs } from "./data";
// console.log(imgs);
export default {
  data() {
    return {
      ctx: null,
      cacheNum: 5,
      canvasInfo: {
        w: 500,
        h: 500,
      },
      currentIndex: 0,
      posInfo: {
        startX: 0,
        startY: 0,
        imgW: 200,
        imgH: 400,
        moveX: 1,
        imgsRightX: 0,
      },
      imgList: [],
      currentLastImgItem: null, // 当前批次加载后最后一张图片
    };
  },
  watch: {
    // 监听当前批次最后一张图片的startX,当startX<=canvas的宽时(即最后一张图片刚露头),加载下一批
    currentLastImgItem: {
      handler(n) {
        // console.log(n);
        if (n.startX <= this.canvasInfo.w) {
          console.log(n);
        }
      },
      deep: true,
    },
  },
  created() {
    this.imgList = new Array(imgs.length);
  },
  mounted() {
    this.init();
  },
  methods: {
    init() {
      let canvas = this.$refs.canvas;
      this.ctx = canvas.getContext("2d");
      this.reqImgList();
    },
    async reqImgList() {
      for (let i = 0; i < this.cacheNum; i++) {
        this.imgList[i] = { src: "", data: "", startX: 0 };
        this.imgList[
          i
        ].src = `http://172.28.1.5:88/prod-api/ftpApi/downloadDefect?name=${imgs[i].fileName}&path=${imgs[i].filePath}&access_token=eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9.eyJleHAiOjE2OTg0NzU4OTgsInVzZXJfbmFtZSI6eyJpZCI6ImFkbWluIiwidXNlcm5hbWUiOiJhZG1pbiIsInBob25lTnVtIjpudWxsLCJzeXNVc2VyRmxhZyI6bnVsbCwic2VjdGlvbklkIjoiYWRtaW4iLCJkZXB0SWQiOiI1MzlhZThmOTQzMTE0Zjk5OTBjNjA3ZTIxZGE4ODQ5ZSIsImRlcHRQYXJlbnRJZCI6bnVsbCwiZGVwdE5hbWUiOiLlnLDpk4HmgLvlhazlj7giLCJkZXB0Q29kZSI6IjAxMDEiLCJ1bml0RmxhZyI6bnVsbCwidXNlckRlcHRBdHRyIjp7ImRlcHRUeXBlQ29kZXMiOltdLCJqdUZsYWciOmZhbHNlLCJ0bHpnc0ZsYWciOmZhbHNlLCJqdWRkRmxhZyI6ZmFsc2UsImp1WWRzRmxhZyI6ZmFsc2UsImp1R2RjRmxhZyI6ZmFsc2UsInNlZ21lbnRGbGFnIjpmYWxzZSwia2VTaGlGbGFnIjpmYWxzZSwid29ya3Nob3BGbGFnIjpmYWxzZSwiamN3V29ya3Nob3BGbGFnIjpmYWxzZSwiamN3V29ya0FyZWFGbGFnIjpmYWxzZSwid29ya0FyZWFGbGFnIjpmYWxzZX0sInN0YWZmSWQiOm51bGwsInN0YWZmTmFtZSI6IueuoeeQhuWRmCIsInJvbGVzIjpbImFkbWluIl0sImFwcENvZGVzIjpbIkxEIiwiTURDUyIsIkMzNjAiLCJTRCIsIlNERyIsIlhKIiwiR0oiLCJHUCIsIkdEIiwiQzQiLCJHVyIsIkpRUiIsIkhZMzYwIl0sInJvbGVJbmRleFBhZ2VzIjpbbnVsbF19LCJqdGkiOiJmZmMyN2FkNi03MWQ0LTRiNWEtOWNkMC1iNTgzOWFjNTVjMjIiLCJjbGllbnRfaWQiOiJHUCIsInNjb3BlIjpbInJlYWQiLCJ3cml0ZSIsInRydXN0Il19.KLf2i5F61ELYLySHGir2pA6vcdfEJ4b6091hfhNxUGOCoz9RjKv9LuAN88oeroh52Uuc1UfAbuQW6LsISF9uUD2NNOIDIYxffZw9-OBDar--Znv1-cKXC1avuJpeIASAPWfsD5U8dl5tihgXeAK91bpKZuJqGNBnRN8v86oDcCudTogQipp5mOiTdSgDWC1EtLKmO7zNsB6tJ92fGspghYte3ZZlHLCwUJympRInp1Tu201c0zhgjOUgkxrfxwXkKPKjGrpTCMetLU_J2NH5pUI_3JLhalAJX4lDtK4bcqsC44B1Iag7dsClE9RWBzmqqBVBq6sMzG26D8zSfyR8fA`;
        // console.log(i);
        let imgData = await this.reqImg(this.imgList[i]);
        this.imgList[i].data = imgData;
        this.imgList[i].startX = this.posInfo.imgW * i;
        this.imgList[i].move = this.imgAnimate.bind(null, this.imgList[i]);
        this.draw(this.imgList[i].data);
      }
      this.currentLastImgItem = this.imgList[this.cacheNum - 1];

      //   this.animate();
    },
    reqImg(item) {
      return new Promise((resolve) => {
        let img = new Image();
        img.src = item.src;
        img.onload = () => {
          resolve(img);
        };
      });
    },
    draw(img) {
      this.ctx.drawImage(
        img,
        this.posInfo.startX + this.currentIndex * this.posInfo.imgW,
        this.posInfo.startY,
        this.posInfo.imgW,
        this.posInfo.imgH
      );
      this.currentIndex++;
    },
    imgAnimate(item) {
      this.ctx.clearRect(item.startX, 0, this.posInfo.imgW, this.posInfo.imgH);
      this.ctx.drawImage(
        item.data,
        item.startX,
        0,
        this.posInfo.imgW,
        this.posInfo.imgH
      );
      item.startX -= this.posInfo.moveX;
    },
    animate() {
      this.imgList.forEach((item) => {
        item.move && item.move();
      });
      requestAnimationFrame(this.animate);
    },
  },
};
</script>

<style lang="scss" scoped>
.canvas-box {
  display: flex;
  flex-direction: column;
  align-items: flex-start;
  canvas {
    width: 500px;
    height: 500px;
    border: 1px solid cadetblue;
  }
  img {
    width: 200px;
    // height: 120px;
    object-fit: contain;
  }
}
</style>