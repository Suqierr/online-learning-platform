<template>
  <div class="main-content">
    <div style="width: 50%; margin: 30px auto; min-height: 1000px">
      <div style="text-align: center">
        <el-button type="success" style="background-color:#FE7843; border-color:#FE7843">{{ courseData.type === 'VIDEO'? '视频课' : '图文课' }}</el-button>
        <span style="font-size: 20px; font-weight: 550; color: #333333; margin-left: 20px">{{ courseData.name }}</span>
      </div>
      <div style="text-align: center; margin-top: 15px">
        <span style="color: #666666; margin-left: 50px">发布时间：{{ courseData.time }}</span>
      </div>
      <div>
        <div style="font-size: 18px; margin: 10px 0; font-weight: bold; color: #333333">课程资料</div>
        <video :src="courseData.video" v-if="courseData.type === 'VIDEO'" controls style="width: 100%; height: auto"></video>
        <div style="margin-top: 10px; color: #333333">资料链接：<a :href="courseData.file" target="_blank">{{ courseData.file }}</a></div>
      </div>
      <!--   课程介绍区域   -->
      <div style="margin-top: 20px">
        <div style="font-size: 18px; font-weight: bold; color: #333333">课程介绍</div>
        <div style="margin-top: 10px; color: #333333" v-html="courseData.content" class="w-e-text w-e-text-container"></div>
      </div>
      <div style="margin-top: 50px; font-size: 18px">欢迎发表您宝贵的意见</div>
      <div style="margin-top: 20px">
        <el-input type="textarea" :rows="5" v-model="content"></el-input>
      </div>
      <div style="margin-top: 10px; text-align: right">
        <el-button type="primary" @click="submit(content, 0)">提交</el-button>
      </div>
      <div style="margin-top: 30px; margin-bottom: 500px">
        <el-row v-for="item in commentData" style="margin-bottom: 30px">
          <el-col :span="4">
            <div style="display: flex; align-items: center;">
              <img :src="item.userAvatar" alt="" style="width: 50px; height: 50px; border-radius: 50%">
              <div style="flex: 1; margin-left: 10px">{{item.userName}}</div>
            </div>
          </el-col>
          <el-col :span="20">
            <div style="height: 50px; line-height: 50px">
              <el-row>
                <el-col :span="18" style="white-space: nowrap; overflow: hidden; text-overflow: ellipsis;">{{item.content}}</el-col>
                <el-col :span="6" style="text-align: right">{{item.time}}</el-col>
              </el-row>
            </div>
            <div v-for="child in item.children" style="margin-bottom: 5px">
              <el-row>
                <el-col :span="5">
                  <div style="display: flex; align-items: center;">
                    <img :src="child.userAvatar" alt="" style="width: 50px; height: 50px; border-radius: 50%">
                    <div style="flex: 1; margin-left: 5px; ">{{ child.userName }} 回复：</div>
                  </div>
                </el-col>
                <el-col :span="13" style="height: 50px; line-height: 50px; white-space: nowrap; overflow: hidden; text-overflow: ellipsis;">{{child.content}}</el-col>
                <el-col :span="6" style="height: 50px; line-height: 50px; text-align: right">{{child.time}}</el-col>
              </el-row>
            </div>
            <div style="margin-top: 20px">
              <el-input style="width: 400px" v-model="item.tmp"></el-input>
              <el-button type="primary" style="margin-left: 5px" @click="submit(item.tmp, item.id)">回复</el-button>
            </div>
          </el-col>
        </el-row>
      </div>
    </div>
  </div>
</template>
<script>
import E from 'wangeditor'
export default {

  data() {
    let courseId = this.$route.query.id
    return {
      user: JSON.parse(localStorage.getItem('xm-user') || '{}'),
      courseId: courseId,
      courseData: {},
      flag: false,
      content: null,
      commentData: []
    }
  },
  mounted() {
    this.loadCourse()

    this.loadComment()
  },
  // methods：本页面所有的点击事件或者其他函数定义区
  methods: {
    loadCourse() {
      this.$request.get('/course/selectById/' + this.courseId).then(res => {
        if (res.code === '200') {
          this.courseData = res.data
        } else {
          this.$message.error(res.msg)
        }
      })
    },
    submit(content, parentId) {
      let data = {
        userId: this.user.id,
        courseId: this.courseId,
        content: content,
        parentId: parentId,
      }
      this.$request.post('/comment/add', data).then(res => {
        if (res.code === '200') {
          this.$message.success('评论成功')
          this.content = null
          this.loadComment()
        } else {
          this.$message.error(res.msg)
        }
      })
    },
    loadComment() {
      this.$request.get('/comment/selectAll',
          {
        params: {
          courseId: this.courseId
        }
      }
      ).then(res => {
        if (res.code === '200') {
          this.commentData = res.data
        } else {
          this.$message.error(res.msg)
        }
      })
    },
  }
}
</script>