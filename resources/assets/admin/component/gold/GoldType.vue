<style lang="css" module>
    .container {
        padding: 15px;
    }
    .loadding {
        text-align: center;
        font-size: 42px;
    }
    .loaddingIcon {
        animation-name: "TurnAround";
        animation-duration: 1.4s;
        animation-timing-function: linear;
        animation-iteration-count: infinite;
    }
</style>

<template>
    <div :class="$style.container">
        <div v-show="message.success" class="alert alert-success alert-dismissible" role="alert">
            <button type="button" class="close" @click.prevent="offAlert">
                <span aria-hidden="true">&times;</span>
            </button>
            {{ message.success }}
        </div>
        <div v-show="message.error" class="alert alert-danger alert-dismissible" role="alert">
            <button type="button" class="close" @click.prevent="offAlert">
                <span aria-hidden="true">&times;</span>
            </button>
            {{ message.error }}
        </div>
        <div class="panel panel-default">
          <!-- 添加广告 -->
          <div class="panel-heading">
            <router-link class="btn btn-primary btn-sm" to="gold/types/add">添加</router-link>
          </div>
          <!-- 广告列表 -->
          <div class="panel-body">
            <table class="table table-striped">
                <thead>
                    <tr>
                        <th>ID</th>
                        <th>名称</th>
                        <th>单位</th>
                        <th>状态</th>
                        <th>操作</th>
                    </tr>
                </thead>
                <tbody>
                    <tr v-show="loadding">
                        <!-- 加载动画 -->
                        <td :class="$style.loadding" colspan="5">
                            <span class="glyphicon glyphicon-refresh" :class="$style.loaddingIcon"></span>
                        </td>
                    </tr>
                      <!-- 数据存在 -->
                      <template v-if="types.length">
                        <tr v-for="type in types">
                          <td>{{ type.id }}</td>
                          <td>{{ type.name }}</td>
                          <td>{{ type.unit }}</td>
                          <td>
                             <span :class="type.status ? 'text-success' : 'text-danger'">{{ type.status ? '启动' : '关闭' }}</span>
                          <td>
                          <template v-if="type.status !== 1">
                            <button class="btn btn-primary btn-sm" @click.prevent="changeStatus(type.id)">{{ !type.status ? '启动' : '关闭' }}</button>
                            <button class="btn btn-danger btn-sm" @click.prevent="delType(type.id)">删除</button>
                          </template>
                          </td>
                        </tr>
                      </template>
                      <template v-else-if="!loadding">
                        <!-- 数据为空 -->
                        <tr><td colspan="7" style="text-align:center;">无数据</td></tr>
                      </template>
                </tbody>
            </table>
          </div>
        </div>
    </div>
</template>
<script>
import request, { createRequestURI } from '../../util/request';
const HomeComponent = {
    data: () => ({

      loadding: true,

      typs:[],
    
      message: {
        error: null,
        success: null,
      }
    
    }),

    methods: {

      getGoldTypes () {

        this.types = [];
        this.loadding = true;
        
        request.get(
          createRequestURI('gold/types'),
          { validateStatus: status => status === 200 }
        ).then(response => {
          
          this.types = response.data;
          this.loadding = false;

        }).catch(({ response: { data: { errors = ['获取金币类型失败'] } = {} } = {} }) => {

          this.loadding = false;
          this.message.error = errors;

        });

      },

      changeStatus (id) {
        request.patch(
          createRequestURI(`gold/types/${id}/open`),
          { validateStatus: status => status === 201 }
        ).then(({ data: { message: [ message ] = [] } }) => {

            this.getGoldTypes();
            this.message.success = message;

        }).catch(({ response: { data: { errors = ['加载认证类型失败'] } = {} } = {} }) => {

          this.loadding = false;
          this.message.error = errors;

        });
      },

      delType (id) {
        request.delete(
          createRequestURI(`gold/types/${id}`),
          { validateStatus: status => status === 204 }
        ).then(({ data: { message: [ message ] = [] } }) => {

            this.getGoldTypes();
            this.message.success = '删除成功';

        }).catch(({ response: { data: { errors = ['删除失败'] } = {} } = {} }) => {

          this.message.error = errors;

        });
      },

      offAlert () {
        let msg = this.message;
        msg.error = msg.success = null;
      } 
    },

    created () {

      this.getGoldTypes();

    },
};

export default HomeComponent;
</script>
