<template>
  <div class="index-wrapper">
    <div class="navigation-wrapper">
      <div class="navigation-title">
        阅读
      </div>
      <div class="navigation-sub-title">
        清风不识字，何故乱翻书
      </div>
      <div class="search-wrapper">
        <el-input
          size="mini"
          placeholder="搜索书籍"
          v-model="search"
          class="search-input"
        >
          <i slot="prefix" class="el-input__icon el-icon-search"></i>
        </el-input>
      </div>
      <div class="recent-wrapper">
        <div class="recent-title">
          最近阅读
        </div>
        <div class="reading-recent">
          <el-tag
            type="warning"
            class="recent-book"
            @click="
              toDetail(
                readingRecent.url,
                readingRecent.name,
                readingRecent.chapterIndex
              )
            "
            :class="{ 'no-point': readingRecent.url == '' }"
          >
            {{ readingRecent.name }}
          </el-tag>
        </div>
      </div>
      <div class="setting-wrapper">
        <div class="setting-title">
          基本设定
        </div>
        <div class="setting-item">
          <el-tag
            :type="connectType"
            class="setting-connect"
            :class="{ 'no-point': newConnect }"
            @click="setIP"
          >
            {{ connectStatus }}
          </el-tag>
        </div>
      </div>
      <div class="bottom-icons">
        <a href="https://github.com/zsakvo/web-yuedu" target="_blank">
          <div class="bottom-icon">
            <img :src="require('../assets/imgs/github.png')" alt="" />
          </div>
        </a>
      </div>
    </div>
    <div class="shelf-wrapper" ref="shelfWrapper">
      <div class="books-wrapper">
        <div class="wrapper">
          <div
            class="book"
            v-for="book in shelf"
            :key="book.noteUrl"
            @click="toDetail(book.bookUrl, book.name, book.durChapterIndex)"
          >
            <div class="cover-img">
              <img
                class="cover"
                :src="book.coverUrl || require('../assets/imgs/noCover.jpeg')"
                alt=""
              />
            </div>
            <div
              class="info"
              @click="toDetail(book.bookUrl, book.name, book.durChapterIndex)"
            >
              <div class="name">{{ book.name }}</div>
              <div class="sub">
                <div class="author">
                  {{ book.author }}
                </div>
                <div class="dot">•</div>
                <div class="size">共{{ book.totalChapterNum }}章</div>
              </div>
              <div class="dur-chapter">已读：{{ book.durChapterTitle }}</div>
              <div class="last-chapter">
                最新：{{ book.latestChapterTitle }}
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import "../assets/fonts/shelffont.css";
import Axios from "axios";

export default {
  data() {
    return {
      search: "",
      readingRecent: {
        name: "尚无阅读记录",
        url: "",
        chapterIndex: 0
      }
    };
  },
  mounted() {
    //获取最近阅读书籍
    let readingRecentStr = localStorage.getItem("readingRecent");
    if (readingRecentStr != null) {
      this.readingRecent = JSON.parse(readingRecentStr);
      if (typeof this.readingRecent.chapterIndex == "undefined") {
        this.readingRecent.chapterIndex = 0;
      }
    }
    if (this.shelf.length == 0) {
      if (localStorage.url) {
        this.loading = this.$loading({
          target: this.$refs.shelfWrapper,
          lock: true,
          text: "正在获取书籍信息",
          spinner: "el-icon-loading",
          background: "rgb(247,247,247)"
        });
        const that = this;
        Axios.get("http://" + localStorage.url + "/getBookshelf", {
          timeout: 3000
        })
          .then(function(response) {
            that.loading.close();
            that.$store.commit("setConnectType", "success");
            that.$store.commit("increaseBookNum", response.data.data.length);
            that.$store.commit(
              "addBooks",
              response.data.data.sort(function(a, b) {
                var x = a["durChapterTime"] || 0;
                var y = b["durChapterTime"] || 0;
                return y - x;
              })
            );
            that.$store.commit(
              "setConnectStatus",
              "已连接 " + localStorage.url
            );
            that.$store.commit("setNewConnect", false);
          })
          .catch(function(error) {
            that.loading.close();
            that.$store.commit("setConnectType", "danger");
            that.$store.commit("setConnectStatus", "点击设置后端 url 与 端口");
            that.$message.error("后端连接失败");
            that.$store.commit("setNewConnect", false);
            throw error;
          });
      } else {
        this.$message.error("请先设置后端 url 与端口");
        this.$store.commit("setConnectStatus", "点击设置后端 url 与 端口");
        this.$store.commit("setNewConnect", false);
        this.$store.commit("setConnectType", "danger");
      }
    }
  },
  methods: {
    setIP() {
      const that = this;
      this.$prompt("请输入 IP 和端口 ( 如：127.0.0.1:9527 )", "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        inputPattern: /^((2[0-4]\d|25[0-5]|[1]?\d\d?)\.){3}(2[0-4]\d|25[0-5]|[1]?\d\d?):([1-9]|[1-9][0-9]|[1-9][0-9][0-9]|[1-9][0-9][0-9][0-9]|[1-6][0-5][0-5][0-3][0-5])$/,
        inputErrorMessage: "url 形式不正确",
        beforeClose: (action, instance, done) => {
          if (action === "confirm") {
            that.$store.commit("setNewConnect", true);
            instance.confirmButtonLoading = true;
            instance.confirmButtonText = "校验中……";
            Axios.get("http://" + instance.inputValue + "/getBookshelf", {
              timeout: 3000
            })
              .then(function(response) {
                instance.confirmButtonLoading = false;
                that.$store.commit(
                  "increaseBookNum",
                  response.data.data.length
                );
                that.$store.commit("addBooks", response.data.data);
                that.$store.commit("setConnectType", "success");
                that.$store.commit(
                  "setConnectStatus",
                  "已连接 " + localStorage.url
                );
                that.$store.commit("setNewConnect", false);
                done();
              })
              .catch(function(error) {
                instance.confirmButtonLoading = false;
                instance.confirmButtonText = "确定";
                that.$message.error("访问失败，请检查您输入的 url");
                that.$store.commit("setNewConnect", false);
                throw error;
              });
          } else {
            done();
          }
        }
      })
        .then(({ value }) => {
          localStorage.url = value;
          this.$message({
            type: "success",
            message: "与" + value + "连接成功"
          });
        })
        .catch(() => {});
    },
    toDetail(bookUrl, bookName, chapterIndex) {
      sessionStorage.setItem("bookUrl", bookUrl);
      sessionStorage.setItem("bookName", bookName);
      sessionStorage.setItem("chapterIndex", chapterIndex);
      this.readingRecent = {
        name: bookName,
        url: bookUrl,
        chapterIndex: chapterIndex
      };
      localStorage.setItem("readingRecent", JSON.stringify(this.readingRecent));
      this.$router.push({
        path: "/chapter"
      });
    }
  },
  computed: {
    shelf() {
      return this.$store.state.shelf;
    },
    connectStatus() {
      return this.$store.state.connectStatus;
    },
    connectType() {
      return this.$store.state.connectType;
    },
    newConnect() {
      return this.$store.state.newConnect;
    }
  }
};
</script>

<style lang="stylus" scoped>
.index-wrapper {
  height: 100%;
  width: 100%;
  display: flex;
  flex-direction: row;

  .navigation-wrapper {
    width: 260px;
    min-width: 260px;
    padding: 48px 36px;
    background-color: #F7F7F7;

    .navigation-title {
      font-size: 24px;
      font-weight: 500;
      font-family: FZZCYSK;
    }

    .navigation-sub-title {
      font-size: 16px;
      font-weight: 300;
      font-family: FZZCYSK;
      margin-top: 16px;
      color: #b1b1b1;
    }

    .search-wrapper {
      .search-input {
        border-radius: 50%;
        margin-top: 24px;

        >>> .el-input__inner {
          border-radius: 50px;
          border-color: #E3E3E3;
        }
      }
    }

    .recent-wrapper {
      margin-top: 36px;

      .recent-title {
        font-size: 14px;
        color: #b1b1b1;
        font-family: FZZCYSK;
      }

      .reading-recent {
        margin: 18px 0;

        .recent-book {
          font-size: 10px;
          // font-weight: 400;
          // margin: 12px 0;
          // font-weight: 500;
          // color: #6B7C87;
          cursor: pointer;
          // padding: 6px 18px;
        }
      }
    }

    .setting-wrapper {
      margin-top: 36px;

      .setting-title {
        font-size: 14px;
        color: #b1b1b1;
        font-family: FZZCYSK;
      }

      .no-point {
        pointer-events: none;
      }

      .setting-connect {
        font-size: 8px;
        margin-top: 16px;
        // color: #6B7C87;
        cursor: pointer;
      }
    }

    .bottom-icons {
      position: fixed;
      bottom: 0;
      height: 120px;
      width: 260px;
      align-items: center;
      display: flex;
      flex-direction: row;
    }
  }

  .shelf-wrapper {
    padding: 48px 48px;
    width: 100%;

    >>>.el-icon-loading {
      font-size: 36px;
      color: #B5B5B5;
    }

    >>>.el-loading-text {
      font-weight: 500;
      color: #B5B5B5;
    }

    .books-wrapper {
      height: 100%;
      overflow: scroll;

      .wrapper {
        display: grid ;
        grid-template-columns: repeat(auto-fill, 380px);
        justify-content: space-around;
        grid-gap: 10px;

        .book {
          user-select: none;
          display: flex;
          cursor: pointer;
          margin-bottom: 18px;
          padding: 24px 24px;
          width: 360px;
          flex-direction: row;
          justify-content: space-around;

          .cover-img {
            width: 84px;
            height: 112px;

            .cover {
              width: 84px;
              height: 112px;
            }
          }

          .info {
            display: flex;
            flex-direction: column;
            justify-content: space-around;
            align-items: left;
            height: 112px;
            margin-left: 20px;
            flex: 1;

            .name {
              width: fit-content;
              font-size: 16px;
              font-weight: 700;
              color: #33373D;
            }

            .sub {
              display: flex;
              flex-direction: row;
              font-size: 12px;
              font-weight: 600;
              color: #6b6b6b;

              .dot {
                margin: 0 7px;
              }
            }

            .intro, .dur-chapter, .last-chapter {
              color: #969ba3;
              font-size: 13px;
              margin-top: 3px;
              font-weight: 500;
              word-wrap: break-word;
              overflow: hidden;
              text-overflow: ellipsis;
              display: -webkit-box;
              -webkit-box-orient: vertical;
              -webkit-line-clamp: 1;
              text-align: left;
            }
          }
        }

        .book:hover {
          background: rgba(0, 0, 0, 0.1);
          transition-duration: 0.5s;
        }
      }

      .wrapper:last-child {
        margin-right: auto;
      }
    }

    .books-wrapper::-webkit-scrollbar {
      width: 0 !important;
    }
  }
}
</style>
