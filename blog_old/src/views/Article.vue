<template>
  <div id="left">
    <el-card class="box-card">
        <div slot="header" class="clearfix">
            <h3><i class="iconfont icon-wenzhang"></i>文章列表</h3>
            <h3 style="float:right">共{{count}}篇</h3>
        </div>
        <div class="content" v-for="(item) in List" :key="item.id">
            <div class="img">
                <img :src="item.img" @click="detailPage(item.article_id)" alt="图片">
            </div>   
            <div class="text">
                <h2 class="title"><i class="iconfont icon-biezhen"></i> {{item.title}}</h2>
                <p class="desc"><span style="font-size:16px;font-weight:bold">文章简介：</span>{{item.brief}}</p>
                <div class="text-bottom">
                    <div class="creater">
                        <div class="img">
                        <img 
                            :src="item.avatar" 
                            alt="">
                        </div>
                        <span class="name"><i class="el-icon-user"></i>{{item.name}}</span>
                        <span class="time"><i class="el-icon-time"></i>{{item.create_time | formateDate}}</span>
                        <!-- <span @click="changeLike(item.like_Star, item.id, index)"><i class="iconfont icon-dianzan"></i>{{item.like_Star}}</span> -->
                        <span><i class="iconfont icon-pinglun"></i>{{item.replyCount}} 条评论</span>
                        <span><i class="iconfont icon-zhiboguankanshu"></i>{{item.visited}}</span>
                    </div>
                    <div class="tag">
                        <el-tag><i class="iconfont icon-biaoqian" style="font-size:.8rem;margin-right:.5rem;"></i>{{item.label}}</el-tag>
                    </div>
                </div>
            </div>
        </div>
        <div class="paginationWrap">
            <!-- <span>共 {{count}} 篇</span> -->
            <el-pagination
                class="pagination"
                background
                layout="total, prev, pager, next"
                :current-page='page'
                :page-size='pageSize'
                @current-change='handlePageChange'
                :total="count">
            </el-pagination>
        </div>

    </el-card>
  </div>
</template>

<script>
import eventBus from '../utils/eventBus'
export default {
    data() {
        return {
            // 总文章数
            count: 0,
            // 文章列表
            List: [],
            // 每页数量
            pageSize: 3,
            // 页数
            page: 1,
            // 点赞数量数组
            likes: [],
            // 标识是否为Category组件传来的
            isCategory: false,
            // 标识是否为Tag组件传来的
            isTag: false
        }
    },
    mounted() {
        this.PageChange(1)
    },
    beforeUpdate() {
        // 这里的this是项目vue实例，用that接受，与eventBus的vue区分
        const that = this
        eventBus.$on('eventFromCategory', function(val) {
            // console.log(val)
            that.List = val.List || []
            that.count = val.count
            that.page = val.page
            that.isCategory = true
            that.isTag = false
            // that.pageSize = val.length
        })
        eventBus.$on('eventFromTag', function(val) {
            // console.log(val)
            that.List = val.List || []
            that.count = val.count
            that.page = val.page
            that.isCategory = false
            that.isTag = true
            // that.pageSize = val.length
        })
        eventBus.$on('eventFromSearch', function(val) {
            that.List = val.List || []
            that.count = val.count
        })
    },
    methods: {
        // 跳转详情页
        detailPage(article_id) {
            this.$router.push(`/detail/${article_id}`)
        },
        // 换页的回调
        handlePageChange(val) {
            this.page = val;
            // console.log(val)
            if (this.isCategory) {
                // console.log('cate')
                this.emitToCategory();
            } else if (this.isTag) {
                // console.log('tag')
                this.emitToTag()
            } else {
                this.PageChange(val)
            }
        },
        // 获取文章列表
        async PageChange(index) {
            try {
                this.$store.commit('LoadingTitleChange', {isShow: true, title: '正在加载文章内容,请稍等...'})
                const res = await this.$api.getArticle(index, this.pageSize)
                // console.log(res)
                if (res.code === 200) {
                    this.List = res.data.data
                    this.count = res.data.count
                    this.List.map((item) => {
                        this.likes.push(item.like_Star)
                    })
                } else {
                    this.$message.error('网络出错了,(ノへ￣、)！')
                }
                this.$store.commit('LoadingTitleChange', {isShow: false, title: ''})
            } catch (error) {
                this.$message.error(error)
            }
        },
        // 点赞
        async changeLike(like_Star, id, index) {
            try {
                if (localStorage.getItem('token')) {
                    if (localStorage.getItem(`like${id}`)) {
                        this.$message.error("你已经为这篇文章点过赞了噢~o(*￣▽￣*)o")
                    } else {
                        localStorage.setItem(`like${id}`, id)
                        this.List.forEach(async (item) => {
                            if(item.id == id) {
                                item.like_Star +=1;
                                const res = await this.$api.getLike({
                                    likestar: item.like_Star,
                                    id: id
                                })
                                if (res.err === 0) {
                                    this.$message.success("你为这篇文章增加了一个star谢谢你的支持鸭！(●ˇ∀ˇ●)")
                                } else {
                                    this.$message.error("网络好像有点差劲呢！小主稍后再来咱们不急！(ノへ￣、)")
                                }
                            }
                        })
                    }
                } else {
                    this.$message.error("请先去登陆再来点赞噢小主！(ノへ￣、)")
                }
            } catch (error) {
                this.$message.error(error)
            }
            
        },
        // 传页数到分类组件
        async emitToCategory(page, pageSize) {
            eventBus.$emit('eventToCategory', {
                page: this.page,
                pageSize: this.pageSize
            })
        },
        // 传页数到标签组件
        async emitToTag(page, pageSize) {
            eventBus.$emit('eventToTag', {
                page: this.page,
                pageSize: this.pageSize
            })
        },
        // 传页数到搜索组件
        async emitToSearch(page, pageSize) {
            
        }
    }
}
</script>

<style lang='scss' scoped>
#left{
    // width: 100%;
  .box-card {
    width: 100%;
    background-color: rgba($color: #f2f2f2, $alpha: 0.6);
    h3{
        display: inline;
    }
    .content{
        display: flex;
        flex-direction: row;
        flex-wrap: wrap;
        padding-bottom: 1.25rem;
        border-bottom: 1px solid #f3f3f3;
        margin-bottom: 1.25rem;
        .img{
            width:15.625rem;
            margin-right: 1.875rem;
            border-radius: .5rem;
            overflow: hidden;
            cursor: pointer;
            // transition: all 1s;
            :hover{
                transition: all .3s;
                position: relative;
                top: -0.1875rem;
                left: -0.09375rem;
                box-shadow: 0rem 5px 10px 3px #ccc;
            }
            img{
                width: 100%;
            }
        }
        .text{
            width: 32rem;
            min-height: 14rem;
            position: relative;
            .title{
                display: block;
                margin-bottom: 1.25rem;
            }
            .desc{
                height: 6.25rem;
                font-size: 16px;
            }
            .text-bottom{
                width: 100%;
                position: absolute;
                bottom: 0;
                .creater{
                    margin-bottom: 1rem;
                    display: flex;
                    // justify-content: center;
                    align-items: center;
                    flex-wrap: wrap;
                    font-size: 14px;
                    i{
                        margin-right: 8px;
                    }
                    span{
                        font-size: 16px;
                        font-weight: bold;
                        margin-left: .5rem;
                        margin-right: .5rem;
                    }
                    .img{
                        // display: block;
                        width: 3rem;
                        height: 3rem;
                        border-radius: 50%;
                        img{
                            width: 100%;
                            height: 100%;
                        }
                    }
                    .time{
                        display: block;
                        // flex: 1;
                        // width: 13.5rem;
                    }
                }   
            }

        }
    }
    @media screen and (max-width: 768px) {
        .content{
            .img{
                width: 100%;
                margin-right: 0;
                margin: .8rem;
            }
            .text{
                .text-bottom{
                    position: relative;
                }
            }
        }
    }
  }
}
@media screen and (max-width: 768px) {
    #left{
        width: 100%;
        // margin: 1rem 0;
    }
}
.paginationWrap{
    
    .pagination{
        float: right;
        margin-bottom: 1rem;
    }

}

</style>