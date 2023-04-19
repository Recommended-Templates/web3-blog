<template>
  <div class="home" v-if="config">
    <div class="head">
      <div
        class="banner"
        :style="
          config.Banner[0].value
            ? 'background-image: url(' + config.Banner[0].value + ');'
            : ''
        "
      ></div>
      <div class="info">
        <div class="container">
          <div class="avatar">
            <img
              :src="
                config.Avatar[0].value
                  ? config.Avatar[0].value
                  : require('@/assets/head.png')
              "
              alt=""
            />
          </div>
          <div class="media-box">
            <span v-for="item in mediaList" :key="item.name">
              <img
                v-if="item.link"
                :src="item.icon"
                alt=""
                @click="goMedia(item.link)"
              />
            </span>
          </div>
          <div class="nickname">{{ config.Name[0].value }}</div>
          <div class="desc">
            {{ config.Bio[0].value }}
          </div>
        </div>
      </div>
    </div>
    <div class="main">
      <div class="container">
        <div class="vlog" v-if="config.YouTube[1].value">
          <div class="title">{{ config.YouTube[0].value || "YouTube" }}</div>
          <iframe
            width="1040"
            height="500"
            :src="formatYoutube(config.YouTube[1].value)"
            title="YouTube video player"
            frameborder="0"
            allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share"
            allowfullscreen
          ></iframe>
        </div>
        <div class="flex-block">
          <div class="mirror" v-if="config.Mirror[1].value">
            <div class="title">{{ config.Mirror[0].value || "Mirror" }}</div>
            <div class="mirror-content">
              <div
                class="mirror-item"
                v-for="item in mirrorList"
                :key="item.guid"
                @click="goMedia(item.link)"
              >
                <div class="mirror-title">{{ item.title }}</div>
                <div class="mirror-date">{{ dateChange(item.pubDate) }}</div>
                <div
                  class="mirror-desc"
                  v-html="getFirstPText(item.encoded.toString())"
                ></div>
                <div class="mirror-info">
                  <span
                    ><img
                      class="mirror-heade"
                      :src="mirrorObj.rss.channel.image.url"
                      alt=""
                  /></span>
                  <span class="mirror-nickname">{{
                    mirrorObj.rss.channel.title.replace(" — Mirror", "")
                  }}</span>
                  <span class="mirror-hash">
                    {{
                      mirrorObj.rss.channel.link
                        .replace("https://mirror.xyz/", "")
                        .substring(0, 6)
                    }}
                  </span>
                </div>
              </div>
            </div>
          </div>
          <div class="twitter" v-if="config.Twitter[1].value">
            <div class="title">{{ config.Twitter[0].value || "Twitter" }}</div>
            <div class="twitter-box">
              <blockquote class="twitter-tweet">
                <a class="twitter-timeline" :href="config.Twitter[1].value"></a>
              </blockquote>
            </div>
          </div>
        </div>
        <div
          class="empty"
          v-if="
            !config.YouTube[1].value &&
            !config.Mirror[1].value &&
            !config.Twitter[1].value
          "
        >
          <img src="@/assets/empty.png" alt="" />
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import x2js from "x2js";

export default {
  name: "HomeView",
  components: {},
  data() {
    return {
      config: null,
      mediaList: [
        {
          name: "Twitter",
          link: "",
          icon: require("@/assets/twitter.png"),
        },
        {
          name: "Discord",
          link: "",
          icon: require("@/assets/discord.png"),
        },
        {
          name: "Telegram",
          link: "",
          icon: require("@/assets/telegram.png"),
        },
        {
          name: "Mirror",
          link: "",
          icon: require("@/assets/mirror.png"),
        },
        {
          name: "Github",
          link: "",
          icon: require("@/assets/github.png"),
        },
        {
          name: "Instagram",
          link: "",
          icon: require("@/assets/ins.png"),
        },
        {
          name: "Facebook",
          link: "",
          icon: require("@/assets/facebook.png"),
        },
      ],
      mirrorObj: null,
      mirrorList: [],
    };
  },
  created() {},
  mounted() {
    this.getConfig();
  },
  methods: {
    getConfig() {
      this.axios
        .get("/config.json")
        .then((res) => {
          const { data } = res;
          this.config = data;
          if (data.Mirror[1].value) {
            this.getMirror(data.Mirror[1].value);
          }
          this.mediaList.forEach((item) => {
            let key = item.name;
            data[key].forEach((item2) => {
              if (item2.key == "link") {
                return (item.link = item2.value);
              }
            });
          });
        })
        .catch((error) => {
          console.log(error);
        });
    },
    getMirror(link) {
      const $x2js = new x2js();
      const url = link.replace(
        "https://submirror.xyz",
        "https://web3.4everblog.org"
      );
      this.axios
        .get(url)
        .then((res) => {
          const { data } = res;
          const mirrorObj = $x2js.xml2js(data);
          this.mirrorObj = mirrorObj;
          this.mirrorList = mirrorObj.rss.channel.item;
          console.log(mirrorObj);
        })
        .catch((error) => {
          console.log(error);
        });
    },
    getFirstPText(html) {
      const pRegex = /<p>(.*?)<\/p>/g;
      const pMatch = pRegex.exec(html);
      if (pMatch != null) {
        let firstPText = pMatch[1];
        if (firstPText.includes("<img")) {
          firstPText = "";
        }
        return firstPText;
      }
    },
    dateChange(dateStr) {
      const date = new Date(dateStr);
      return date.toLocaleDateString("en-US", {
        year: "numeric",
        month: "long",
        day: "numeric",
      });
    },
    goMedia(link) {
      window.open(link);
    },
    formatYoutube(link) {
      const nStr = "https://www.youtube.com/watch?v=";
      const mStr = "https://youtu.be/";
      let v = "";
      let n = link.indexOf(nStr);
      let m = link.indexOf(mStr);
      if (n != -1) {
        v = link.replace(nStr, "");
        return "https://www.youtube.com/embed/" + v;
      } else if (m != -1) {
        v = link.replace(mStr, "");
        return "https://www.youtube.com/embed/" + v;
      } else {
        return link;
      }
    },
  },
};
</script>
<style lang="less">
.container {
  max-width: 1040px;
  margin: 0 auto;
}
.home {
  background-color: #f9fbfc;
  padding-bottom: 100px;
  .head {
    .banner {
      background-image: url("@/assets/banner.png");
      width: 100%;
      height: 220px;
      background-repeat: no-repeat;
      background-size: cover;
      background-position: center;
    }
    .info {
      position: relative;
      background-color: #fff;
      padding-top: 10px;
      min-height: 180px;
      box-shadow: 0px 4px 12px 0px rgba(0, 0, 0, 0.1);
      .avatar {
        width: 102px;
        height: 102px;
        line-height: 108px;
        background: #ffffff;
        border-radius: 50%;
        position: absolute;
        top: -54px;
        text-align: center;
        padding: 6px;
        overflow: hidden;
        img {
          width: 96px;
          height: 96px;
          border-radius: 50%;
        }
      }
      .media-box {
        display: flex;
        align-items: center;
        justify-content: flex-end;
        height: 50px;
        img {
          width: 24px;
          margin: 10px;
          cursor: pointer;
        }
      }
      .nickname {
        font-size: 28px;
        color: #0b0817;
        font-weight: bold;
        overflow: hidden;
        text-overflow: ellipsis;
        white-space: nowrap;
      }
      .desc {
        font-size: 16px;
        font-weight: 400;
        color: #6c7789;
        margin-top: 10px;
        display: -webkit-box;
        -webkit-box-orient: vertical;
        -webkit-line-clamp: 3;
        overflow: hidden;
      }
    }
  }
}
.vlog {
  margin-top: 40px;
}
.flex-block {
  display: flex;
  margin-top: 70px;
  .mirror {
    min-width: 625px;
    margin-right: 20px;
    width: 100%;
  }
  .twitter {
    min-width: 400px;
    width: 100%;
    .twitter-box {
      width: 100%;
      max-height: 1200px;
      overflow: scroll;
    }
  }
}
.title {
  font-size: 28px;
  font-weight: bold;
  color: #0b0817;
  margin-bottom: 20px;
}
.empty {
  text-align: center;
  margin-top: 175px;
  img {
    width: 300px;
  }
}
.mirror-content {
  display: flex;
  flex-wrap: wrap;
  justify-content: space-between;
  max-height: 1200px;
  overflow: scroll;
  .mirror-item {
    width: 300px;
    background: #ffffff;
    box-shadow: 0px 4px 12px 0px rgba(0, 0, 0, 0.05);
    border-radius: 10px;
    margin: 10px 0;
    padding: 0 20px;
    padding-bottom: 20px;
    box-sizing: border-box;
    cursor: pointer;
    .mirror-title {
      min-height: 100px;
      padding: 22px 0;
      font-size: 20px;
      font-weight: bold;
      display: flex;
      align-items: center;
    }
    .mirror-date {
      font-size: 14px;
      font-weight: 500;
      color: #d0dae9;
      line-height: 20px;
    }
    .mirror-desc {
      display: -webkit-box;
      -webkit-box-orient: vertical;
      -webkit-line-clamp: 3;
      overflow: hidden;
      font-size: 14px;
      font-weight: 500;
      color: #6c7789;
      min-height: 60px;
      margin: 10px 0;
    }
    .mirror-info {
      display: flex;
      align-items: center;
      .mirror-heade {
        width: 24px;
        border-radius: 50%;
      }
      .mirror-nickname {
        font-size: 14px;
        font-weight: 500;
        color: #0b0817;
        margin-left: 10px;
      }
      .mirror-hash {
        width: 62px;
        background: #ffffff;
        border-radius: 11px;
        border: 1px solid #889ab3;
        font-size: 10px;
        font-weight: 500;
        color: #889ab3;
        margin-left: 10px;
        text-align: center;
      }
    }
  }
}
</style>
