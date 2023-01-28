<script setup lang="ts">
import { computed, reactive, ref, type Ref } from "vue";
import axios from "axios";
import moment from 'moment';

interface REPO {
  id: number;
  full_name: string;
}

const username = ref("");
const TOKEN = ref("")
let repos: Ref = ref(Array<REPO>);

function getRepos() {
  const url = `/api/users/${username.value}/repos?per_page=100`;
  axios.get(url).then((res) => {
    repos.value = res.data;
  });
}

function deleteRepo(repoName: string) {
  let _confirm = confirm("是否删除 " + repoName);
  const url = `/api/repos/${repoName}`;
  const headers = {
    Authorization: `token ${TOKEN.value}`,
  };
  if (_confirm == true) {
    axios.delete(url, {
      headers: headers
    }).then((res) => {
      console.log(res)
      if (res.status == 204) {
        alert("删除成功")
      }
    }).catch((err) => {
      console.log(err)
      alert(err.response.data.message)
    })
  }
}

function reactiveTime(time: string) {
  return moment(time).fromNow()
}

</script>

<template>
  <div class="container">
    <div class="ui message  red">
      请提前获取TOKEN用于操作，点击删除后<b>无法恢复</b>请谨慎操作
    </div>
    <div>
      <div class="ui input">
        <input type="password" v-model="TOKEN" placeholder="TOKEN">
      </div>
      <div class="ui action input">
        <input type="text" v-model="username" placeholder="用户名">
        <button @click="getRepos" class="ui teal left labeled icon button">
          <i class="search icon"></i>
          获取仓库
        </button>
      </div>
      <div class="ui relaxed divided list repo" v-for="repo in repos" key="repo.id">
        <div class="item">
          <button class="ui icon red button right floated" @click="deleteRepo(repo.full_name)">
            <i class="trash alternate icon"></i>
            {{ repo.full_name }}
          </button>
          <i class="big github middle aligned icon"></i>
          <div class="content">
            <a class="header" :href="repo.html_url">{{ repo.full_name }}</a>
            <div class="icons">
              <i class="code branch icon"></i> <b>{{ repo.forks_count }}</b>
              &nbsp;&nbsp;
              <i class="eye icon"></i> <b>{{ repo.watchers }}</b>
              &nbsp;&nbsp;
              <i class="clock outline icon"></i> <b>{{ reactiveTime(repo.updated_at) }}</b>
            </div>
            <div class="content grey">
              {{ repo.description }}
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped lang="scss">
.container {
  height: 100vh;
  width: 100vw;
  box-sizing: border-box;
  padding: 20px;

  .repo {
    border-radius: 8px;
    border: 1px solid rgba(255, 255, 255, 1);
    padding: 10px;

    &:hover {
      border: 1px solid rgba(0, 0, 0, 0.4);
      background: rgba(0, 0, 0, 0.1);
    }
  }
}
</style>
