<template>
  <div class="home-page">
    <section class="welcome-card">
      <div class="welcome-copy">
        <span class="eyebrow">WELCOME TO DeerVison</span>
        <h2>欢迎回来，{{ $auth.user().name }}</h2>
        <p>今天也要保持从容与高效，开始处理您的工作事项吧。</p>
        <div class="user-meta">
          <span><a-icon type="user" /> {{ $auth.user().name }}</span>
          <span><a-icon type="mail" /> {{ $auth.user().email }}</span>
        </div>
      </div>
      <div class="welcome-visual">
        <div class="visual-ring visual-ring--outer"></div>
        <div class="visual-ring visual-ring--inner"></div>
        <div class="visual-icon"><a-icon type="rise" /></div>
      </div>
    </section>

    <section class="overview-card">
      <div class="section-heading">
        <div>
          <h3>业务概览</h3>
          <p>集中管理核心业务，让每一步进展都有迹可循</p>
        </div>
        <span class="status"><i></i> 系统运行正常</span>
      </div>
      <div class="feature-grid">
        <div class="feature-item feature-item--blue">
          <span class="feature-icon"><a-icon type="profile" /></span>
          <div><strong>订单管理</strong><p>跟进订单与交付进度</p></div>
          <a-icon class="feature-arrow" type="arrow-right" />
        </div>
        <div class="feature-item feature-item--green">
          <span class="feature-icon"><a-icon type="file-text" /></span>
          <div><strong>稿件管理</strong><p>归档并维护稿件资料</p></div>
          <a-icon class="feature-arrow" type="arrow-right" />
        </div>
        <div class="feature-item feature-item--purple">
          <span class="feature-icon"><a-icon type="bar-chart" /></span>
          <div><strong>数据统计</strong><p>掌握团队业务表现</p></div>
          <a-icon class="feature-arrow" type="arrow-right" />
        </div>
      </div>
    </section>
  </div>
</template>

<style lang="less" scoped>
.home-page {
  display: grid;
  gap: 14px;
}

.welcome-card,
.overview-card {
  border: 1px solid #e0e9f4;
  border-radius: 12px;
  background: #fff;
  box-shadow: 0 7px 25px rgba(32, 78, 130, 0.05);
}

.welcome-card {
  position: relative;
  display: flex;
  min-height: 290px;
  align-items: center;
  justify-content: space-between;
  padding: 46px 58px;
  overflow: hidden;
  background: linear-gradient(125deg, #fff 0%, #f5f9ff 54%, #eaf3ff 100%);

  &::before {
    content: "";
    position: absolute;
    right: -110px;
    bottom: -170px;
    width: 430px;
    height: 430px;
    border: 64px solid rgba(47, 124, 246, 0.05);
    border-radius: 50%;
  }

  .welcome-copy {
    position: relative;
    z-index: 1;

    .eyebrow {
      color: #2f7cf6;
      font-size: 11px;
      font-weight: 700;
      letter-spacing: 1.8px;
    }

    h2 {
      margin: 12px 0 10px;
      color: #1b345a;
      font-size: 32px;
      font-weight: 700;
    }

    > p {
      margin: 0;
      color: #7f90aa;
      font-size: 15px;
    }
  }

  .user-meta {
    display: flex;
    flex-wrap: wrap;
    gap: 10px;
    margin-top: 30px;

    span {
      padding: 8px 13px;
      border: 1px solid #dce8f7;
      border-radius: 20px;
      color: #58708f;
      background: rgba(255, 255, 255, 0.75);
      font-size: 12px;

      .anticon {
        margin-right: 5px;
        color: #2f7cf6;
      }
    }
  }
}

.welcome-visual {
  position: relative;
  width: 220px;
  height: 220px;
  flex-shrink: 0;

  .visual-ring {
    position: absolute;
    border: 1px solid rgba(47, 124, 246, 0.18);
    border-radius: 50%;

    &--outer {
      inset: 0;
      border-style: dashed;
    }

    &--inner {
      inset: 28px;
      background: rgba(47, 124, 246, 0.05);
    }
  }

  .visual-icon {
    position: absolute;
    top: 50%;
    left: 50%;
    display: flex;
    width: 94px;
    height: 94px;
    align-items: center;
    justify-content: center;
    border-radius: 28px;
    color: #fff;
    background: linear-gradient(145deg, #59a5ff, #276fe9);
    box-shadow: 0 24px 45px rgba(43, 116, 231, 0.28);
    font-size: 42px;
    transform: translate(-50%, -50%) rotate(-5deg);

    .anticon {
      transform: rotate(5deg);
    }
  }
}

.overview-card {
  padding: 24px;
}

.section-heading {
  display: flex;
  align-items: center;
  justify-content: space-between;
  margin-bottom: 20px;

  h3 {
    margin: 0 0 4px;
    color: #263b5e;
    font-size: 17px;
  }

  p {
    margin: 0;
    color: #91a0b5;
    font-size: 12px;
  }

  .status {
    color: #648078;
    font-size: 12px;

    i {
      display: inline-block;
      width: 7px;
      height: 7px;
      margin-right: 5px;
      border-radius: 50%;
      background: #34c69b;
      box-shadow: 0 0 0 4px rgba(52, 198, 155, 0.12);
    }
  }
}

.feature-grid {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 14px;
}

.feature-item {
  position: relative;
  display: flex;
  min-height: 100px;
  align-items: center;
  gap: 14px;
  padding: 18px;
  border: 1px solid #e4ecf6;
  border-radius: 10px;
  background: #fbfdff;
  transition: all 0.2s ease;

  &:hover {
    border-color: #cfe0f7;
    box-shadow: 0 10px 24px rgba(39, 86, 143, 0.08);
    transform: translateY(-2px);
  }

  .feature-icon {
    display: flex;
    width: 44px;
    height: 44px;
    flex-shrink: 0;
    align-items: center;
    justify-content: center;
    border-radius: 11px;
    color: #2f7cf6;
    background: #e8f2ff;
    font-size: 20px;
  }

  strong {
    color: #324b6e;
    font-size: 14px;
  }

  p {
    margin: 5px 0 0;
    color: #98a6ba;
    font-size: 12px;
  }

  .feature-arrow {
    margin-left: auto;
    color: #afbdd0;
  }

  &--green .feature-icon {
    color: #18a887;
    background: #def6ef;
  }

  &--purple .feature-icon {
    color: #7a5ee9;
    background: #eee9ff;
  }
}

@media (max-width: 900px) {
  .welcome-card {
    padding: 34px;
  }

  .welcome-visual {
    display: none;
  }

  .feature-grid {
    grid-template-columns: 1fr;
  }
}
</style>
