# 系统功能

## 1. 系统概述

- 本项目为微信小程序 + 微信云开发（Cloud Functions + 云数据库）的资讯系统。
- 业务核心围绕“NBA赛事 + 资讯 + 数据查询 + 社区互动 + 运营统计”的闭环能力。
- 小程序端通过 `wx.cloud.callFunction` 调用云函数实现数据读写与权限控制。

## 2. 小程序端（用户侧）功能

### 2.1 首页（`pages/index/index`）

- **Feed 模式切换（推荐/关注）**
  - 推荐模式：按栏目分类浏览资讯。
  - 关注模式：展示关注的社区作者动态、资讯作者文章、关注实体相关内容（云函数 `getFollowFeed`）。
- **关注页横滑栏**
  - 横向滚动展示关注的资讯作者、社区作者、球队/球员头像卡片，点击跳转对应主页。
- **栏目分类展示与切换**
  - 默认包含"推荐"。
  - 其他栏目从云函数 `getCategories` 获取。
- **资讯列表流**
  - 云函数 `getNewsList`。
  - 支持分页：`skip/limit`。
  - 支持按栏目筛选：`category`。
- **热门阅读轮播（Top Read）**
  - 云函数 `getTopReadNews`（按 `viewCount` 排序）。
- **焦点赛事**
  - 展示正在进行/即将开始的赛事卡片，点击跳转赛事详情。
- **首页底部定制区**
  - 关注球队下一场比赛卡片。
  - 关注球队相关资讯列表。
  - 最近浏览记录。
  - 聚合云函数 `getHomeBlocks` 一次性返回。
- **下拉刷新 / 触底加载更多**
- **搜索入口**
  - 输入关键词后跳转搜索页 `pages/search/search`。

### 2.2 搜索页（`pages/search/search`）

- **搜索框**
  - 支持输入关键词后点击搜索或按回车确认搜索。
- **历史搜索记录**
  - 本地存储最近15条搜索关键词，支持一键清空。
  - 点击历史记录快速搜索。
- **热搜词**
  - 预置NBA热门搜索词（球队、球员、赛事），点击快速搜索。
- **关键词搜索**
  - 调用 `getNewsList`，传入 `keyword`。
- **搜索结果分页**
  - `skip/limit`，支持下拉刷新、触底加载。
- **点击进入详情页**

### 2.3 详情页（`pages/detail/detail`）

- **资讯详情展示**
  - 云函数 `getNewsDetail` 返回 `detail`。
  - 同时返回是否已收藏 `isFavorited`。
- **阅读量统计**
  - 云函数 `incViewCount` 将 `viewCount` 自增。
- **收藏 / 取消收藏**
  - 云函数 `toggleFavorite`。
- **分享至微信好友/群聊**
  - `onShareAppMessage` + `open-type="share"` 按钮。
- **评论列表展示（抖音风格）**
  - 云函数 `getComments`（仅展示已上架 `status=0` 的评论）。
  - 抖音风格UI：头像居左、堆叠内容、图标操作栏、可折叠回复、相对时间显示。
  - 支持楼中楼回复（`addReply` / `getReplies`）。
  - 回复显示「用户A ▸ 用户B」格式，标明回复对象（`replyToName`）。
  - 评论和回复均支持点赞（♥），云函数 `toggleCommentLike`。
- **评论/回复举报**
  - 长按评论/回复可举报，调用 `reportContent` 云函数。
- **发表评论**
  - 云函数 `addComment`。
  - 新评论默认 `status=2`（待审核）。
  - 评论成功后通知资讯发布者（写入 `notifications`，type=`comment`）。
- **资讯点赞**
  - 云函数 `toggleNewsLike`，操作 `news_likes` 集合。

### 2.4 我的（`pages/user/user`）

- **用户资料展示**
  - 云函数 `getMyProfile` 获取当前用户信息与 `isAdmin`。
- **授权登录（获取头像昵称）**
  - 优先使用 `wx.getUserProfile` 自动获取微信头像昵称。
  - 若获取到默认灰色头像，自动 fallback 到 `open-type="chooseAvatar"` + `type="nickname"` 手动选择模式。
  - 云函数 `updateUserProfile` 写入/更新用户资料。
- **快捷入口**
  - **消息中心**：展示未读消息数量红点，点击进入消息中心页面。
  - 个人信息、数据查询、反馈。
- **我的关注**
  - 合并展示三种关注类型：用户关注（`user_follows`）、作者关注（`author_follows`）、球队/球员关注（`follows`）。
- **浏览记录**
  - 云函数 `getHistory`。
- **我的收藏**
  - 云函数 `getFavorites`。
- **初始化演示数据**
  - 云函数 `initDemoData`。
- **管理员入口（仅管理员可用）**
  - 新闻发布管理、评论管理、栏目管理。

### 2.5 赛事页

- **赛程/比分列表**
  - 按“正在进行 / 即将开始 / 已结束”展示。
- **赛事筛选与检索**
  - 支持按日期、球队等更细粒度筛选。
- **赛事详情**
  - 展示对阵双方、节比分、状态等信息。
  - 技术统计面板：当 `match.stats` 存在时展示，不存在时自动隐藏。

### 2.6 数据查询

- **球员数据检索**（得分/篮板/助攻等）
- **球队信息与战绩**
- **排名（standings）**
- **历史战绩**
  - 在赛事详情中展示双方最近交手记录。

### 2.7 个性化服务

- **关注球队/球员**
- **专属资讯**
  - 基于关注关键词的简易推荐。
- **历史浏览记录**

### 2.8 球迷社区

- **社区动态**
  - 发布动态 + 列表展示。
  - 发帖自动审核（敏感词/广告/频控检测，云函数 `addPost`）。
- **话题/讨论区**
  - 话题列表、按话题筛选、发帖关联话题。
  - 用户可自定义创建话题（云函数 `createTopic`，含敏感词检测与微信内容安全检测）。
- **帖子互动**
  - 帖子详情页：评论 + 楼中楼回复。
  - 支持评论点赞、删除自己的评论（长按删除）。
  - 支持举报帖子评论/回复（`reportContent`）。
  - **评论回复折叠**：超过3条回复自动折叠，点击"展开剩余N条回复"查看全部，支持收起。
- **帖子分享**
  - 帖子详情页支持分享至微信好友/群聊（`onShareAppMessage`）。

### 2.9 反馈

- **提交反馈**
  - 用户可提交反馈。
- **反馈处理**
  - 管理员可查看、标记处理并回复。

### 2.10 赛事提醒

- **预约赛事提醒**
  - 用户在赛事详情页点击「预约提醒」按钮。
  - 授权订阅消息后，系统记录预约。
- **开赛前提醒发送**
  - 云函数 `sendMatchReminders` 定时检查即将开赛的比赛。
  - 向已预约用户发送微信订阅消息通知。
  - 支持管理端手动触发。
- **配置要求**
  - 需要在微信公众平台申请订阅消息模板。
  - 管理员在「数据维护」页面配置模板ID后生效。
  - 详细配置步骤见 `赛事提醒配置指南.md`。

### 2.11 消息中心（`pages/messages/messages`）

- **消息通知列表**
  - 展示所有通知，按时间倒序排列。
  - 支持下拉刷新、触底加载更多。
- **通知类型**
  - **评论通知**：有人评论了你的资讯/帖子（`type=comment`）。区分资讯评论和动态评论。
  - **回复通知**：有人回复了你的评论（`type=reply`）。
  - **点赞通知**：有人赞了你的评论（`type=like`），显示被赞内容预览。
  - **关注通知**：有人关注了你（`type=follow`）。
- **未读红点**
  - 在「我的」页面的「消息中心」快捷入口显示未读消息数量红点。
  - 云函数 `getUnreadCount` 查询未读数。
- **自动标记已读**
  - 进入消息中心页面后自动调用 `markNotificationsRead` 将所有未读消息标记为已读。
- **消息跳转**
  - 点击关注通知跳转到关注者的用户主页。
  - 点击评论/回复/点赞通知跳转到对应资讯或帖子详情页。

### 2.12 用户投稿（`pages/submit-news/submit-news`）

- **投稿发布**
  - 用户可投稿资讯文章（标题、摘要、正文、分类、封面图）。
  - 云函数 `submitNews`，投稿默认 `auditStatus=pending`（待审核）。
- **文件/网页导入**
  - 支持上传 Word 文档（.docx），云函数 `parseDocument` 解析为 HTML 正文。
  - 支持输入网页 URL，云函数 `parseWebpage` 抓取并提取文章内容。

### 2.13 帖子点赞

- 云函数 `togglePostLike`，操作 `post_likes` 集合。

## 3. 管理端（管理员侧）功能

> 管理端云函数统一通过 `users.isAdmin === true` 做权限校验（`assertAdmin`）。

### 3.1 新闻发布与管理（`pages/admin/publish/publish`）

- **新增 / 编辑新闻（Upsert）**
  - 云函数 `adminUpsertNews`。
  - 字段包含：`title/author/summary/content/coverImg/category/publishDate/viewCount`。
- **封面图上传**
  - 使用 `wx.cloud.uploadFile` 上传到云存储，保存 `fileID` 到 `coverImg`。
- **新闻列表（后台）**
  - 云函数 `adminListNews`。
- **删除新闻**
  - 云函数 `adminDeleteNews`。

### 3.2 评论管理（`pages/admin/comments/comments`）

- **查看评论列表**
  - 云函数 `adminListComments`。
- **上架评论**
  - 云函数 `adminSetCommentStatus`（设置 `status=0`）。
- **下架/驳回评论**
  - 支持填写驳回原因，写入 `rejectReason`、`rejectTime`、`auditTime` 字段。
- **删除评论**
  - 云函数 `adminDeleteComment`。

### 3.2.1 举报管理（`pages/admin/reports/reports`）

- **举报列表**
  - 云函数 `getReports`，支持按状态筛选（全部/待处理/已处理）。
- **处理举报**
  - 云函数 `handleReport`，支持驳回或删除被举报内容。
  - 删除操作同时将被举报内容标记为 `status=3`。

### 3.3 栏目（分类）管理（`pages/admin/category/category`）

- **新增栏目 / 更新排序**
  - 云函数 `adminUpsertCategory`。
- **栏目列表**
  - 云函数 `adminListCategories`。
- **启用 / 停用栏目**
  - 云函数 `adminSetCategoryActive`。
- **删除栏目**
  - 云函数 `adminDeleteCategory`。
  - 删除时会把该栏目下新闻批量回退到“推荐”。
  - “推荐”栏目不可删除。

### 3.4 赛事数据维护

- **赛程/比分/状态维护（CRUD）**
- **球员/球队/排名维护（CRUD）**
- **比分修正与异常处理**
  - 支持通过修改 `matches.status/startTime/score/note` 实现延期/取消/比分修正。

### 3.5 用户管理（`pages/admin/users/users`）

- **用户列表**
  - 云函数 `adminListUsers`。
- **用户封禁/禁言**
  - 云函数 `adminSetUserStatus`。

### 3.5.1 反馈管理（`pages/admin/feedback/feedback`）

- **反馈列表**
  - 云函数 `adminListFeedback`。
- **标记处理**
  - 云函数 `adminSetFeedbackStatus`。
- **回复反馈**
  - 云函数 `adminReplyFeedback`。

### 3.5.2 社区帖子审核（`pages/admin/community-posts/community-posts`）

- **帖子列表**
  - 云函数 `adminListPosts`。
- **上架/下架帖子**
  - 云函数 `adminSetPostStatus`，支持驳回原因、审核时间。
- **删除帖子**
  - 云函数 `adminDeletePost`。

### 3.5.3 社区评论审核（`pages/admin/community-comments/community-comments`）

- **评论列表**
  - 云函数 `adminListPostComments`。
- **上架/下架评论**
  - 云函数 `adminSetPostCommentStatus`，支持驳回原因、审核时间。
- **删除评论**
  - 云函数 `adminDeletePostComment`。

### 3.6 运营数据统计

- **运营数据看板**
  - 用户总数/今日新增/近7天新增
  - 新闻总数/上架数/下架数/总阅读量
  - 收藏总数/今日收藏
  - 评论总数/待审核评论
  - 反馈总数/未处理反馈
  - 赛程条目数
- **日报/趋势图**
  - 近7天/30天趋势
  - 周/月汇总与环比
- **导出能力**
  - 支持导出 CSV（复制到剪贴板）。
- **历史口径回填工具**
  - 管理员一键回填 `posts/post_comments` 的 `day` 字段，用于补齐历史趋势。

### 3.7 投稿审核（`pages/admin/review-news/review-news`）

- **投稿列表**
  - 云函数 `adminReviewNews`（action=`list`），支持按状态筛选（待审/已通过/已驳回）。
- **审核操作**
  - 云函数 `adminReviewNews`（action=`approve`/`reject`），通过或驳回用户投稿。

### 3.8 赛季配置与批量导入/同步

- **当前赛季配置**
  - `settings.currentSeason`
- **一键导入数据（傻瓜化）**
  - 在「赛事维护」页面，选择爬虫生成的 JSON 文件（news.json、teams.json、players.json、matches.json、standings.json）。
  - 系统自动从文件名识别数据类型，客户端读取并分批导入，无需手动配置。
  - 支持单个文件导入和多个文件全部导入。
  - 导入失败时弹窗显示详细错误原因。
- **资讯批量导入**
  - 云函数 `adminImportNews`，按标题去重，支持批量 upsert 资讯到 `news` 集合。
- **赛季数据批量导入（JSON/云文件）**
  - 支持 dryRun 预检查 + 批量 upsert。
- **赛季数据包同步（ETL）**
  - fileID 单包同步：downloadFile → normalize → upsert。
  - 同步日志写入 `season_sync_logs`。
- **第三方 API 同步（通用适配器）**
  - 通过 `settings.officialApiConfig` 配置启用/关闭与映射规则。

## 4. 云函数（后端能力）清单

### 4.1 用户与权限

- **`login`**
  - 小程序启动调用，确保 `users` 集合存在基础用户记录。
- **`getMyProfile`**
  - 返回用户信息与 `isAdmin`。
- **`updateUserProfile`**
  - 更新 `nickName/avatarUrl`。

### 4.1.1 用户辅助能力

- **`getFollows`**：获取关注列表。
- **`getHistory`**：获取浏览历史。
- **`getPersonalizedNews`**：基于关注关键词获取专属资讯。

### 4.2 新闻

- **`getNewsList`**：按 `category/keyword` 获取新闻列表，支持分页。
- **`getNewsDetail`**：获取新闻详情，并返回当前用户是否收藏。
- **`getTopReadNews`**：获取阅读量最高的新闻。
- **`incViewCount`**：阅读量自增。

### 4.2.1 阅读与统计辅助

- **`news_views` 记录**：用于运营统计与日报聚合（配合 `generateDailyReports`）。

### 4.3 收藏

- **`toggleFavorite`**：收藏/取消收藏切换。
- **`getFavorites`**：获取我的收藏列表。

### 4.4 评论

- **`addComment`**：新增评论（默认待审核 `status=2`）。
- **`getComments`**：获取已上架评论列表（`status=0`）。

### 4.4.1 评论互动

- **`toggleCommentLike`**：评论/回复点赞/取消点赞，操作 `comment_likes` 集合。点赞时写入通知（`notifications`，type=`like`）。
- **`deleteMyComment`**：删除自己的评论。

### 4.5 分类

- **`getCategories`**：获取启用状态的分类列表。

### 4.6 管理员功能

- **新闻**：`adminListNews` / `adminUpsertNews` / `adminDeleteNews`
- **分类**：`adminListCategories` / `adminUpsertCategory` / `adminSetCategoryActive` / `adminDeleteCategory`
- **评论**：`adminListComments` / `adminSetCommentStatus` / `adminDeleteComment`

### 4.6.1 管理端扩展

- **赛事维护**：`adminListMatches` / `adminUpsertMatch` / `adminDeleteMatch`
- **球员/球队/排名**：`adminListPlayers` / `adminUpsertPlayer` / `adminDeletePlayer` / `adminListTeams` / `adminUpsertTeam` / `adminDeleteTeam` / `adminListStandings` / `adminUpsertStanding` / `adminDeleteStanding`
- **用户管理**：`adminListUsers` / `adminSetUserStatus`
- **反馈**：`adminListFeedback` / `adminReplyFeedback` / `adminSetFeedbackStatus`
- **统计**：`adminGetStats` / `generateDailyReports`
- **回填工具**：`adminBackfillUserJoinDay` / `adminBackfillCommunityDay`
- **导入/同步**：赛季数据导入/同步相关云函数（支持 dryRun 与 upsert）
- **资讯导入**：`adminImportNews`（批量导入爬虫资讯）
- **投稿审核**：`adminReviewNews`（审核用户投稿）
- **文档/网页解析**：`parseDocument` / `parseWebpage`

### 4.6.2 社区

- **发帖/列表**：`addPost` / `getPosts` / `getPostDetail`
- **帖子评论**：`addPostComment` / `getPostComments`
- **回复**：`addReply` / `getReplies`

### 4.6.3 资讯/帖子互动

- **`toggleNewsLike`**：资讯点赞/取消点赞，操作 `news_likes` 集合。
- **`togglePostLike`**：帖子点赞/取消点赞，操作 `post_likes` 集合。
- **`submitNews`**：用户投稿资讯，默认待审核。

### 4.7 演示数据

- **`initDemoData`**：初始化演示数据。

### 4.8 赛事与数据

- **赛事**：`getMatches` / `getMatchDetail`
- **球员/球队/排名**：用户侧查询相关云函数
- **统计数据**：`player_stats` / `team_stats` 的查询能力（配合管理端导入）

### 4.9 赛事提醒

- **订阅**：`subscribeMatchReminder`
- **发送**：`sendMatchReminders`

### 4.10 关注体系

- **球队/球员关注**：`toggleFollow` / `getFollows`
- **社区作者关注**：`toggleUserFollow` / `getMyUserFollows`
- **资讯作者关注**：`toggleAuthorFollow` / `getMyAuthorFollows`
- **关注 Feed 聚合**：`getFollowFeed`（混排社区动态+资讯）

### 4.11 首页定制区

- **`getHomeBlocks`**：聚合返回关注球队下一场比赛、球队相关资讯、最近浏览记录。

### 4.12 举报

- **`reportContent`**：用户举报评论/回复，写入 `reports` 集合，达阈值自动转待审。
- **`getReports`**：管理员获取举报列表。
- **`handleReport`**：管理员处理举报（驳回/删除）。

### 4.13 用户账户

- **`updateMyAccount`**：更新用户信息及微信绑定状态。
- **`unbindWechat`**：解除微信绑定。

## 5. 数据集合（云数据库）与核心字段

- **`users`**
  - `openid`, `nickName`, `avatarUrl`, `isAdmin`, `joinTime`
- **`news`**
  - `title`, `author`, `summary`, `content`, `coverImg`, `category`, `viewCount`, `publishDate`
- **`categories`**
  - `name`, `sort`, `isActive`
- **`favorites`**
  - `_openid`, `newsId`, `title`, `createTime`
- **`comments`**
  - `newsId`, `userInfo{nickName,avatarUrl}`, `content`, `createTime`, `status(0/1/2)`

- **`matches`**
  - `league`, `startTime`, `status`, `homeTeam`, `awayTeam`, `homeScore`, `awayScore`, `periodScores`, `stats`, `updateTime`
- **`teams`**
  - `name`, `conference`, `division`, `wins`, `losses`
- **`players`**
  - `name`, `team`, `ppg`, `rpg`, `apg`
- **`standings`**
  - `conference`, `rank`, `team`, `wins`, `losses`

- **`posts` / `post_comments` / `replies` / `topics`**
  - 社区动态、评论、回复与话题相关数据

- **`feedback`**
  - 用户反馈与管理员回复

- **`daily_reports` / `settings`**
  - 日报聚合数据与系统配置

- **`reports`**
  - 用户举报记录：`targetType`, `targetId`, `reason`, `handleStatus`, `handleAction`

- **`follows` / `user_follows` / `author_follows`**
  - 球队/球员关注、社区作者关注、资讯作者关注

- **`match_subscriptions`**
  - 赛事提醒订阅记录

## 6. 数据采集（Python 爬虫）

- **爬虫模块**（`crawler/nba_crawler.py`）
  - 从 ESPN 公开 API 采集 NBA 球队、球员、排名、赛程和资讯数据。
  - 输出为 JSON 文件，可直接导入云数据库。
  - 内置 fallback 样本数据，确保离线可用。
  - 技术栈：Python 3.8+ / requests / BeautifulSoup4。

## 7. UI 主题

- **NBA 官方红蓝配色**
  - 主色：#17408B（NBA蓝）
  - 辅色：#C9082A（NBA红）
  - 导航栏、TabBar 选中态、按钮、标签、渐变背景等统一使用 NBA 配色。

## 8. 功能闭环检查清单

| 功能模块      | 用户侧                   | 管理端          | 云函数                                                  | 状态           |
| ------------- | ------------------------ | --------------- | ------------------------------------------------------- | -------------- |
| 资讯浏览      | ✅ 首页/搜索/详情         | ✅ 发布/管理     | ✅ getNewsList/getNewsDetail                             | 闭环           |
| 资讯收藏      | ✅ 收藏/取消              | -               | ✅ toggleFavorite/getFavorites                           | 闭环           |
| 资讯评论      | ✅ 发表/列表              | ✅ 审核/删除     | ✅ addComment/getComments                                | 闭环           |
| 资讯分享      | ✅ 分享按钮               | -               | -                                                       | 闭环           |
| 赛事浏览      | ✅ 列表/详情              | ✅ 维护          | ✅ getMatches/getMatchDetail                             | 闭环           |
| 赛事提醒      | ✅ 预约按钮               | ✅ 模板ID配置    | ✅ subscribeMatchReminder/sendMatchReminders             | 闭环（需配置） |
| 数据查询      | ✅ 球员/球队/排名         | ✅ 维护          | ✅ 相关云函数                                            | 闭环           |
| 社区发帖      | ✅ 发布/列表              | ✅ 审核          | ✅ addPost/getPosts                                      | 闭环           |
| 社区评论      | ✅ 评论/回复              | ✅ 审核          | ✅ addPostComment/addReply                               | 闭环           |
| 帖子分享      | ✅ 分享按钮               | -               | -                                                       | 闭环           |
| 举报功能      | ✅ 举报入口               | ✅ 举报列表/处理 | ✅ reportContent/getReports/handleReport                 | 闭环           |
| 关注体系      | ✅ 关注球队/球员/作者     | -               | ✅ toggleFollow/getFollows                               | 闭环           |
| 个性化首页    | ✅ 底部定制区             | -               | ✅ getHomeBlocks                                         | 闭环           |
| 搜索增强      | ✅ 历史/热搜              | -               | -                                                       | 闭环           |
| 消息中心      | ✅ 通知列表/红点/点赞通知 | -               | ✅ getNotifications/getUnreadCount/markNotificationsRead | 闭环           |
| 用户投稿      | ✅ 投稿页面               | ✅ 审核管理      | ✅ submitNews/adminReviewNews                            | 闭环           |
| 文档/网页导入 | ✅ 上传解析               | ✅ 管理端使用    | ✅ parseDocument/parseWebpage                            | 闭环           |
| 资讯/帖子点赞 | ✅ 点赞按钮               | -               | ✅ toggleNewsLike/togglePostLike                         | 闭环           |
| 话题创建      | ✅ 自定义话题             | ✅ 管理          | ✅ createTopic                                           | 闭环           |
| 用户反馈      | ✅ 提交反馈               | ✅ 查看/回复     | ✅ addFeedback/adminListFeedback                         | 闭环           |
| 运营统计      | -                        | ✅ 数据看板      | ✅ adminGetStats/generateDailyReports                    | 闭环           |
| 用户管理      | -                        | ✅ 封禁/禁言     | ✅ adminSetUserStatus                                    | 闭环           |
| 数据采集      | -                        | ✅ 一键导入      | ✅ Python爬虫/adminImportNews                            | 闭环           |

## 9. 待部署云函数清单

以下云函数已编写完成，需通过微信开发者工具右键「上传并部署：云端安装依赖」：

- `getHomeBlocks` — 首页底部定制区聚合
- `getReports` / `handleReport` — 举报管理
- `getFollowFeed` — 关注模式Feed
- `addPost` — 发帖（含自动审核）
- `toggleUserFollow` / `toggleAuthorFollow` — 关注作者
- `subscribeMatchReminder` / `sendMatchReminders` — 赛事提醒
- `adminImportNews` — 资讯批量导入
- `adminReviewNews` — 投稿审核
- `submitNews` — 用户投稿
- `parseDocument` / `parseWebpage` — 文档/网页解析
- `toggleNewsLike` / `togglePostLike` — 资讯/帖子点赞
- `toggleCommentLike` — 评论点赞（含通知写入）
- `addComment` — 评论（含通知写入）
- `getReplies` — 回复列表（含replyToName填充）
- 其他新增云函数

## 10. 相关文档

- `未完成功能模块.md` — 待增强功能清单
- `赛事提醒配置指南.md` — 订阅消息模板ID配置步骤
- `云数据库集合清单.md` — 数据库集合结构（30个集合）
- `云函数清单.md` — 云函数完整清单及功能说明（121个云函数）
- `爬虫使用指南.md` — Python爬虫完整使用指南（功能、实现、导入方法）
- `crawler/README.md` — Python爬虫快速开始

*最后更新：2026年3月*
