<script>
  import Header from './Header.svelte';
</script>

<Header text="Flood Transmission" />
<p>
  Flood 是 Transmission 的替代 Web UI。
  它是一个前端 Web 应用程序，不需要运行任何额外的服务即可与 Transmission 通信。
  尽管它是一项正在进行的工作，但它绝对足够好用。
</p>
<p>
  Flood 是
  <a href="https://github.com/Flood-UI/flood" target="_blank">Flood</a> 的克隆，
  而后者最初是为 rTorrent 构建的。所有设计和功能功劳都归于它的创建者。
  然而，代码根本不是复制的，它是从头开始构建的，以便与 Transmission 配合使用。
</p>

<Header text="Feedback" />
<p>
  如果您有特定问题或错误，请提交
  <a
  href="https://github.com/johman10/flood-for-transmission/issues/new"
  target="_blank">GitHub 问题</a
  >。您也可以通过这种方式提出功能请求。
</p>

<style>
  p {
    color: var(--color-modal-text);
    font-size: 14px;
    margin-bottom: 15px;
  }

  a {
    color: var(--color-modal-link);
    text-decoration: none;
  }
</style>
