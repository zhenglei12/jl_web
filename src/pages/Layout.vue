<template>
  <a-layout class="layout">
    <a-layout-sider
      v-model="collapsed"
      class="sidebar"
      :width="224"
      :collapsed-width="76"
      :trigger="null"
      collapsible
    >
      <div class="logo" @click="goto('/')">
        <span class="logo-mark"><a-icon type="file-text" /></span>
        <span v-if="!collapsed" class="logo-copy">
          <strong>渐鹿管理系统</strong>
          <small>JIANLU SYSTEM</small>
        </span>
      </div>
      <a-menu
        theme="dark"
        mode="inline"
        :open-keys="openKey"
        :selectedKeys="[$route.name]"
        @openChange="openChange"
      >
        <a-sub-menu v-acl:one="['user-list', 'role-list', 'permission-list']" key="user">
          <span slot="title"><a-icon type="user" /><span>用户管理</span></span>
          <a-menu-item v-acl="'user-list'" key="person" @click="goto('/person')">用户列表</a-menu-item>
          <a-menu-item v-acl="'role-list'" key="role" @click="goto('/role')">角色列表</a-menu-item>
          <a-menu-item v-acl="'department-list'" key="department" @click="goto('/department')">部门列表</a-menu-item>
        </a-sub-menu>
        <a-menu-item v-acl="'order-list'" key="task-order" @click="goto('/order')">
          <a-icon type="dollar" />
          <span>订单列表</span>
        </a-menu-item>
        <a-menu-item v-acl="'classify-list'" key="classify" @click="goto('/classify')">
          <a-icon type="hdd" />
          <span>文档库</span>
        </a-menu-item>
        <a-sub-menu
          v-acl:one="[
            'edit-statistics.all.list',
            'edit-statistics.day.list',
            'staff-statistics.list',
            'edit-statistics.order.list',
          ]"
          key="statistic"
        >
          <span slot="title">
            <a-icon type="area-chart" />
            <span>报表管理</span>
          </span>
          <a-menu-item v-acl="'edit-statistics.all.list'" key="statistic-all" @click="goto('/statistic/all')"
            >总览</a-menu-item
          >
          <a-menu-item v-acl="'edit-statistics.day.list'" key="statistic-day" @click="goto('/statistic/day')"
            >日统计</a-menu-item
          >
          <a-menu-item v-acl="'staff-statistics.list'" key="statistic-user" @click="goto('/statistic/user')"
            >员工统计</a-menu-item
          >
          <a-menu-item v-acl="'edit-statistics.order.list'" key="statistic-order" @click="goto('/editorder')"
            >编辑订单列表</a-menu-item
          >
        </a-sub-menu>
        <a-menu-item v-acl="'manuscript_bank-list'" key="manuscript" @click="goto('/manuscript')">
          <a-icon type="book" />
          <span>稿件库</span>
        </a-menu-item>
      </a-menu>
    </a-layout-sider>
    <a-layout>
      <a-layout-header class="header">
        <div class="header-main">
          <a-icon
            class="trigger"
            :type="collapsed ? 'menu-unfold' : 'menu-fold'"
            @click="() => (collapsed = !collapsed)"
          />
          <div class="page-heading">
            <h1>{{ pageTitle }}</h1>
            <p>{{ pageSubtitle }}</p>
          </div>
        </div>
        <div class="user-panel">
          <a-badge dot>
            <a-avatar icon="user" />
          </a-badge>
          <span class="user-name">{{ currentUserName }}</span>
          <a-divider type="vertical" />
          <a-button type="link" class="logout" @click="$auth.logout()">
            <a-icon type="poweroff" />退出
          </a-button>
        </div>
      </a-layout-header>
      <a-layout-content class="content">
        <div class="wraper">
          <router-view />
        </div>
      </a-layout-content>
    </a-layout>
  </a-layout>
</template>

<script>
export default {
  data() {
    return {
      collapsed: false,
      openKey: [],
    };
  },
  mounted() {
    this.syncOpenKey();
  },
  watch: {
    $route() {
      this.syncOpenKey();
    },
  },
  computed: {
    currentUserName() {
      const user = this.$auth.user() || {};
      return user.name || user.username || "管理员";
    },
    pageTitle() {
      const titles = {
        home: "工作台",
        person: "用户管理",
        role: "角色管理",
        department: "部门管理",
        "task-order": "订单管理",
        classify: "文档分类",
        "statistic-all": "数据总览",
        "statistic-day": "日统计",
        "statistic-user": "员工统计",
        "statistic-order": "编辑订单",
        manuscript: "稿件库",
      };
      return titles[this.$route.name] || "渐鹿管理系统";
    },
    pageSubtitle() {
      const subtitles = {
        home: "欢迎回来，开启高效工作的一天",
        person: "统一维护系统用户与账号信息",
        role: "配置角色及对应的数据权限",
        department: "维护组织架构与部门层级",
        "task-order": "高效管理订单，实时掌握交付进度",
        classify: "清晰组织和维护文档分类",
        "statistic-all": "汇总查看编辑任务进展",
        "statistic-day": "按日追踪团队完成情况",
        "statistic-user": "查看客服业绩与回款数据",
        "statistic-order": "查看编辑订单及稿件状态",
        manuscript: "统一管理和下载稿件资料",
      };
      return subtitles[this.$route.name] || "让管理更清晰、更高效";
    },
  },
  methods: {
    syncOpenKey() {
      if (this.$route.meta && this.$route.meta.group) {
        this.openKey = [this.$route.meta.group];
        return;
      }
      this.openKey = this.$route.name && this.$route.name.indexOf("statistic-") === 0 ? ["statistic"] : [];
    },
    goto(path) {
      this.$router.push(path);
    },
    openChange(e) {
      this.openKey = e.filter((_) => _ !== this.openKey[0]);
    },
  },
};
</script>

<style lang="less" scoped>
.layout {
  height: 100vh;
  overflow: hidden;

  .sidebar {
    position: relative;
    z-index: 12;
    overflow: hidden;
    background: linear-gradient(180deg, #0b2346 0%, #0b1d38 62%, #102a55 100%);
    box-shadow: 8px 0 28px rgba(12, 35, 70, 0.12);

    &::after {
      content: "";
      position: absolute;
      right: -70px;
      bottom: -120px;
      width: 330px;
      height: 330px;
      border: 55px solid rgba(47, 111, 237, 0.1);
      border-radius: 50%;
      pointer-events: none;
    }

    /deep/ .ant-layout-sider-children {
      display: flex;
      flex-direction: column;
    }

    /deep/ .ant-menu {
      flex: 1;
      padding: 8px 10px 24px;
      background: transparent;
      border: 0;
    }

    /deep/ .ant-menu-inline,
    /deep/ .ant-menu-vertical,
    /deep/ .ant-menu-vertical-left {
      border-right: 0;
    }

    /deep/ .ant-menu-item,
    /deep/ .ant-menu-submenu-title {
      height: 44px;
      line-height: 44px;
      margin: 4px 0;
      border-radius: 8px;
      color: rgba(232, 241, 255, 0.82);
      transition: all 0.2s ease;
    }

    /deep/ .ant-menu-item:hover,
    /deep/ .ant-menu-submenu-title:hover {
      color: #fff;
      background: rgba(255, 255, 255, 0.08);
    }

    /deep/ .ant-menu-item-selected {
      color: #fff;
      background: linear-gradient(100deg, #2f7cf6 0%, #438fff 100%) !important;
      box-shadow: 0 8px 20px rgba(27, 112, 239, 0.32);
    }

    /deep/ .ant-menu-sub {
      background: rgba(2, 15, 36, 0.22) !important;
      border-radius: 8px;
    }

    /deep/ .anticon {
      font-size: 17px;
    }
  }

  .trigger {
    display: flex;
    width: 34px;
    height: 34px;
    align-items: center;
    justify-content: center;
    margin-right: 14px;
    border-radius: 8px;
    color: #61708a;
    font-size: 17px;
    cursor: pointer;
    transition: all 0.2s ease;

    &:hover {
      color: #2f7cf6;
      background: #edf5ff;
    }
  }

  .logo {
    height: 74px;
    padding: 0 17px;
    display: flex;
    align-items: center;
    gap: 11px;
    color: #fff;
    white-space: nowrap;
    overflow: hidden;
    cursor: pointer;

    &-mark {
      display: inline-flex;
      width: 40px;
      min-width: 40px;
      height: 40px;
      align-items: center;
      justify-content: center;
      border: 1px solid rgba(255, 255, 255, 0.32);
      border-radius: 11px;
      background: linear-gradient(145deg, #55a7ff, #246be5);
      box-shadow: 0 8px 20px rgba(23, 108, 233, 0.35);
      font-size: 21px;
    }

    &-copy {
      display: flex;
      min-width: 0;
      flex-direction: column;
      line-height: 1.2;

      strong {
        font-size: 17px;
        letter-spacing: 1px;
      }

      small {
        margin-top: 5px;
        color: rgba(218, 232, 255, 0.5);
        font-size: 9px;
        letter-spacing: 1.6px;
      }
    }
  }

  .header {
    height: 64px;
    background: #fff;
    padding: 0 24px 0 18px;
    border-bottom: 1px solid #e9eff7;
    box-shadow: 0 4px 18px rgba(40, 72, 120, 0.04);
    z-index: 9;
    display: flex;
    justify-content: space-between;
    align-items: center;

    &-main,
    .user-panel {
      display: flex;
      align-items: center;
    }

    .page-heading {
      display: flex;
      align-items: baseline;
      gap: 14px;

      h1 {
        margin: 0;
        color: #172b4d;
        font-size: 20px;
        font-weight: 700;
        letter-spacing: 0.5px;
      }

      p {
        margin: 0;
        color: #9aa8bd;
        font-size: 12px;
      }
    }

    .user-panel {
      gap: 9px;
      color: #53637c;

      /deep/ .ant-avatar {
        background: linear-gradient(145deg, #5d8ff1, #285fc3);
        box-shadow: 0 4px 10px rgba(41, 101, 204, 0.22);
      }

      /deep/ .ant-badge-dot {
        box-shadow: 0 0 0 2px #fff;
      }

      .user-name {
        max-width: 120px;
        overflow: hidden;
        font-weight: 500;
        text-overflow: ellipsis;
        white-space: nowrap;
      }

      .logout {
        padding: 0 4px;
        color: #7c8ca5;

        &:hover {
          color: #2f7cf6;
        }
      }
    }
  }

  .content {
    padding: 14px;
    overflow: auto;
    background: #f4f8fc;

    .wraper {
      min-height: 100%;
    }
  }
}

@media (max-width: 900px) {
  .layout .header {
    padding-right: 14px;

    .page-heading p,
    .user-name,
    .ant-divider {
      display: none;
    }
  }
}
</style>
