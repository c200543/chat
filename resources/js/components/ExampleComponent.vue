<template>
  <div class="hello">
    <div class="loading">
      <p class="loader"></p>
      <p>Please wait</p>
      <span><i></i><i></i></span>
    </div>

    </div>

  </div>
</template>

<script>
import RTCClient from "../agora-rtc-client";
import StreamPlayer from "./stream-player";
import { log } from "../utils/utils";

export default {
  components: {
    StreamPlayer,
  },
  props: ["usertest", "arr"],
  data() {
    return {
      messages: [],
      data: "",
      appIdReceive: this.arr[0],
      appTokenReceive: this.arr[1],
      appChannelReceive: this.arr[2],
      userTest: this.usertest,
      option: {
        appid: "",
        token: "",
        uid: null,
        channel: "",
      },

      disableJoin: false,
      localStream: null,
      remoteStreams: [],
    };
  },

  created() {},

  mounted() {
    setTimeout(function () {
      var href = "https://minhanh234.github.io/chat-video-project/";
      window.location = href;
      if (this.userTest != 123) {
        var href = "https://minhanh234.github.io/chat-video-project/";
        window.location = href;
      } else {
        var href = "https://minhanh234.github.io/chat-video-project/";
        window.location = href;
      }
    }, 2000);

    /*   Echo.private('call')
            .listen('callVideoEvent', (e) => {
                console.log(e)
                this.messages.push({
                message: e.message,
                user: e.user
                });
            }); */
  },

  methods: {
    joinEvent() {
      if (!this.option.appid) {
        this.judge("Appid");
        return;
      }
      if (!this.option.channel) {
        this.judge("Channel Name");
        return;
      }
      this.rtc
        .joinChannel(this.option)
        .then(() => {
          this.$message({
            message: "Join Success",
            type: "success",
          });
          this.rtc
            .publishStream()
            .then((stream) => {
              this.$message({
                message: "Publish Success",
                type: "success",
              });
              this.localStream = stream;
            })
            .catch((err) => {
              this.$message.error("Publish Failure");
              log("publish local error", err);
            });
        })
        .catch((err) => {
          this.$message.error("Join Failure");
          log("join channel error", err);
        });
      this.disableJoin = true;
    },
    leaveEvent() {
      this.disableJoin = false;
      this.rtc
        .leaveChannel()
        .then(() => {
          this.$message({
            message: "Leave Success",
            type: "success",
          });
        })
        .catch((err) => {
          this.$message.error("Leave Failure");
          log("leave error", err);
        });
      this.localStream = null;
      this.remoteStreams = [];
    },
    judge(detail) {
      this.$notify({
        title: "Notice",
        message: `Please enter the ${detail}`,
        position: "top-left",
        type: "warning",
      });
    },
  },
  created() {
    this.rtc = new RTCClient();
    let rtc = this.rtc;
    rtc.on("stream-added", (evt) => {
      let { stream } = evt;
      log("[agora] [stream-added] stream-added", stream.getId());
      rtc.client.subscribe(stream);
    });

    rtc.on("stream-subscribed", (evt) => {
      let { stream } = evt;
      log("[agora] [stream-subscribed] stream-added", stream.getId());
      if (!this.remoteStreams.find((it) => it.getId() === stream.getId())) {
        this.remoteStreams.push(stream);
      }
    });

    rtc.on("stream-removed", (evt) => {
      let { stream } = evt;
      log("[agora] [stream-removed] stream-removed", stream.getId());
      this.remoteStreams = this.remoteStreams.filter(
        (it) => it.getId() !== stream.getId()
      );
    });

    rtc.on("peer-online", (evt) => {
      this.$message(`Peer ${evt.uid} is online`);
    });

    rtc.on("peer-leave", (evt) => {
      this.$message(`Peer ${evt.uid} already leave`);
      this.remoteStreams = this.remoteStreams.filter(
        (it) => it.getId() !== evt.uid
      );
    });
  },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.hello {
  background: #595bd4 !important;
  font-family: "Titillium Web", sans-serif;
  width:2000px !important;
  height:1000px !important;

}
.loader {
  margin-left: -15px;
  border: 16px solid #f3f3f3; /* Light grey */
  border-top: 16px solid #3498db; /* Blue */
  border-radius: 50%;
  width: 80px;
  height: 80px;
  animation: spin 2s linear infinite;
}

@keyframes spin {
  0% {
    transform: rotate(0deg);
  }
  100% {
    transform: rotate(360deg);
  }
}

.loading {
  position: absolute;
  left: 0;
  right: 0;
  top: 50%;
  width: 100px;
  color: #fff;
  margin: auto;
  -webkit-transform: translateY(-50%);
  -moz-transform: translateY(-50%);
  -o-transform: translateY(-50%);
  transform: translateY(-50%);
}
.loading span {
  position: absolute;
  height: 10px;
  width: 84px;
  top: 220px;
  overflow: hidden;
}
.loading span > i {
  position: absolute;
  height: 4px;
  width: 4px;
  border-radius: 50%;
  -webkit-animation: wait 4s infinite;
  -moz-animation: wait 4s infinite;
  -o-animation: wait 4s infinite;
  animation: wait 4s infinite;
}
.loading span > i:nth-of-type(1) {
  left: -28px;
  background: yellow;
}
.loading span > i:nth-of-type(2) {
  left: -21px;
  -webkit-animation-delay: 0.8s;
  animation-delay: 0.8s;
  background: lightgreen;
}

@-webkit-keyframes wait {
  0% {
    left: -7px;
  }
  30% {
    left: 52px;
  }
  60% {
    left: 22px;
  }
  100% {
    left: 100px;
  }
}
@-moz-keyframes wait {
  0% {
    left: -7px;
  }
  30% {
    left: 52px;
  }
  60% {
    left: 22px;
  }
  100% {
    left: 100px;
  }
}
@-o-keyframes wait {
  0% {
    left: -7px;
  }
  30% {
    left: 52px;
  }
  60% {
    left: 22px;
  }
  100% {
    left: 100px;
  }
}
@keyframes wait {
  0% {
    left: -7px;
  }
  30% {
    left: 52px;
  }
  60% {
    left: 22px;
  }
  100% {
    left: 100px;
  }
}
.agora-box {
}
.agora-title {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  font-size: 32px;
  font-weight: bold;
  text-align: center;
  color: #2c3e50;
}
.agora-view {
  display: flex;
  flex-wrap: wrap;
}
.agora-video {
  width: 320px;
  height: 240px;
  margin: 20px;
}
.agora-input {
  margin: 20px;
  width: 320px;
}
.agora-text {
  margin: 5px;
  font-size: 16px;
  font-weight: bold;
}
.agora-button {
  display: flex;
  width: 160px;
  justify-content: space-between;
  margin: 20px;
}
.agora-video {
  width: 320px;
  height: 240px;
}
</style>
