<template>
  <div class="page">
    <div class="photo-box">
      <div>
        <div class="title">请选择颜色</div>
        <radio-group @change='colorChange'>
          <radio value="1" :checked='color == 1'>彩色</radio>
          <radio value="2" :checked='color == 2'>黑白</radio>
        </radio-group>
      </div>

      <div>
        <div class="title">请选择类型</div>
        <radio-group @change='typeChange'>
          <radio value="png" :checked="type == 'png'">静图</radio>
          <!--          <radio value="jpg" :checked="type == 'jpg'">jpg</radio>-->
          <radio value="gif" :checked="type == 'gif'">动图</radio>
        </radio-group>
      </div>

      <div class="title">请选择图片</div>
      <div class="actions">
        <div class="item" @click="selectPhotos('camera')">
          <img src="/static/icons/camera.png"/>
          <span>拍摄</span>
        </div>

        <div class="item" @click="selectPhotos('album')">
          <img src="/static/icons/album.png"/>
          <span>相册选择</span>
        </div>
      </div>
    </div>

    <div v-if='photo'>
      <div class="title">本地选择的图片</div>
      <div class="img-item">
        <div class="image-box">
          <img :src="photo" alt=""/>
        </div>
      </div>
    </div>

    <!--    <div v-if='result'>-->
    <div class="title">服务器处理后的图片</div>
    <div class="img-item">
      <div class="image-box">
        <img :src="imgData" alt=""/>
      </div>
    </div>
    <!--    </div>-->

    <div class="submit">
      <button type="default" style='margin: 10px;' @click="delPhoto(index)">删除当前已选择的图片</button>
      <button type="primary" style='margin: 10px;' @click="submit">提交</button>
    </div>
  </div>
</template>

<script>
const fs = wx.getFileSystemManager()
export default {
  name: "index",
  data() {
    return {
      photo: null,
      //    result: null,
      imgData: null,
      color: 1,
      type: 'png',
    };
  },
  methods: {
    selectPhotos(type) {
      console.log("select...");
      let that = this;
      if (type === "camera") {
        wx.chooseImage({
          count: 1,
          sizeType: ['compressed'],
          sourceType: ["camera"],
          success(res) {
            console.log(res);
            // tempFilePath可以作为img标签的src属性显示图片
            const tempFilePaths = res.tempFilePaths;
            that.photo = tempFilePaths[0];
          },
        });
      } else if (type === "album") {
        wx.chooseImage({
          count: 1,
          sizeType: ['compressed'],
          sourceType: ["album"],
          success(res) {
            console.log(res);
            // tempFilePath可以作为img标签的src属性显示图片
            const tempFilePaths = res.tempFilePaths;
            that.photo = tempFilePaths[0];
          },
        });
      }
    },
    delPhoto(index) {
      this.photo = null;
      //   this.result = null;
    },

    typeChange(e) {
      console.log(e)

      this.type = e.mp.detail.value;
    },

    colorChange(e) {
      console.log(e)
      this.color = e.mp.detail.value;
    },
    submit() {
      let that = this;
      if (!this.photo) {
        return wx.showToast({
          title: '至少选择一张照片',
          icon: 'none',
          duration: 2000
        });
      }
      //   方法一 获取 base64 图片编码 然后调用后台接口
      //   wx.getFileSystemManager().readFile({
      //     filePath: that.photo, //选择图片返回的相对路径
      //     encoding: "base64", //编码格式
      //     success: (res) => {
      //       //成功的回调
      //       const base64Data = "data:image/png;base64," + res.data;
      //       console.log(base64Data)
      //       // 调用wx.request();
      //       wx.request({
      //         url: 'http://192.168.0.152:9001/asciiimage/image',
      //         data: {
      //           image: base64Data,
      //           color: this.color,
      //           type: this.type,
      //         },
      //         method: 'POST',
      //         success(res) {
      //           // 后台返回新的图片地址或者base64字符串, 展示在前端
      //           console.log("seccuss")
      //           console.log(res)
      //           console.log(res.data)
      //           //that.result = res.photoUrl;
      //           // that.imgData = res.data.replace(/[\r\n]]/g, '');
      //         },
      //         fail(e) {
      //           console.log("fail")
      //           console.log(e)
      //         }
      //       })
      //     },
      //     fail(e) {
      //       console.log(e)
      //     }
      //   });
      // }

      // 方法二 wx.uploadFile
      wx.uploadFile({
        filePath: this.photo,
        name: 'image',
        formData: {
          color: this.color,
          type: this.type,
        },
        method: 'POST',
        header: {
          "Content-Type": "multipart/form-data"
        },
        url: 'http://192.168.0.152:9001/asciiimage/image',
        success(res) {
          // 后台返回新的图片地址或者base64字符串, 展示在前端
          console.log(res.data)
          // const resdata = wx.arrayBufferToBase64(res.data);
          // console.log('resdata', resdata)
          // that.imgData = resdata.replace(/[\r\n]]/g, '');

          that.imgData = 'data:image/png;base64,' + res.data

          fs.writeFile({
            // 写文件 wx.env.USER_DATA_PATH 指定临时文件存入的路径，后面字符串自定义
            filePath: wx.env.USER_DATA_PATH + '/img1.' + that.type,
            data: res.data,
            encoding: 'base64',
            success(res) {
              wx.getSetting({
                success: (res) => {
                  if (res.authSetting.hasOwnProperty('scope.writePhotosAlbum') === false) {
                    wx.showModal({
                      title: '提示',
                      content: '需要获取相册权限',
                      success: function (res) {
                        if (res.confirm) {
                          //  打开授权页面,让用户手动打开权限
                          wx.openSetting({
                            success: function (data) {
                              // 保存到相册
                              wx.saveImageToPhotosAlbum({
                                filePath: wx.env.USER_DATA_PATH + '/img1.' + that.type,
                                success(res) {
                                  console.log("success");
                                },
                                fail(res) {
                                  console.error(res)
                                }
                              })
                            }

                          })
                        }
                      }
                    })
                  } else {
                    // 保存到相册
                    wx.saveImageToPhotosAlbum({
                      filePath: wx.env.USER_DATA_PATH + '/img1.' + that.type,
                      success(res) {
                        console.log("success");
                      },
                      fail(res) {
                        console.error(res)
                      }
                    })
                  }
                }
              })

            },
            fail(res) {
              console.error(res)
            }
          })
        },
        fail(e) {
          console.log(e)
        }
      })
    },
  },
  onShow() {
  },
  onLoad(options) {
  },
};
</script>

<style>
.actions .item {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  padding: 20px;
  border: 1px solid #eee;
}

.item img {
  width: 80px;
  height: 80px;
}

.title {
  text-align: center;
  font-weight: 600;
  line-height: 40px;
}

.image-box {
  padding: 0 20px;

}

radio-group {
  padding: 20px;
}

radio {
  margin-right: 10px;
}
</style>
