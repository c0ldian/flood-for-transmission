<script>
  import Checkbox from '~components/Checkbox';
  import Button from '~components/Button';
  import { modals, torrents, alerts, selectedTorrents } from '~helpers/stores';

  let loading = false;
  let deleteData = false;

  const term = $selectedTorrents.length > 1 ? 'torrent' : 'torrents';

  const handleRemove = () => {
    if (loading) return;

    loading = true;
    if (!$selectedTorrents.length) {
      loading = false;
      alerts.add('至少选择一个种子才能继续', 'negative');
      return;
    }
    torrents
      .remove($selectedTorrents, deleteData)
      .then(() => {
        torrents.set(
          $torrents.filter((t) => !$selectedTorrents.includes(t.id))
        );
        alerts.add(`成功移除 ${$selectedTorrents.length} ${term}`);
        selectedTorrents.clear();
        modals.close();
      })
      .catch(() => {
        alerts.add(`移除 ${term} 失败`, 'negative');
      });
  };
</script>

<h1>Remove torrents</h1>

<div class="content">
  <form on:submit|preventDefault="{handleRemove}">
    <p>您确定要删除 {$selectedTorrents.length} {term} 吗?</p>
    <Checkbox label="删除本地数据" bind:checked="{deleteData}" />
    <div class="button-group">
      <Button priority="tertiary" on:click="{modals.close}">Cancel</Button>
      <Button priority="primary" loading="{loading}" type="submit">
        Remove torrents
      </Button>
    </div>
  </form>
</div>

<style>
  h1 {
    color: var(--color-modal-header);
    font-size: 20px;
    font-weight: 500;
    padding: 20px 25px 0;
  }

  .content {
    overflow-y: auto;
    padding: 20px 25px 20px 25px;
    color: var(--color-modal-text);
  }

  .button-group {
    display: flex;
    justify-content: flex-end;
    margin-top: 25px;
    gap: 10px;
  }

  p {
    font-size: 14px;
    margin-bottom: 15px;
  }
</style>
