<script>
  import Header from './Header.svelte';
  import Icon from '~components/Icon';
  import Input from '~components/Input';
  import InputPath from '~components/InputPath';
  import Checkbox from '~components/Checkbox';
  import Button from '~components/Button';
  import { session, modals, alerts } from '~helpers/stores';
  import {
    SESSION_COLUMN_DOWNLOAD_DIR,
    SESSION_COLUMN_START_ADDED,
    SESSION_COLUMN_RENAME_PARTIAL_FILES,
    SESSION_COLUMN_IDLE_SEEDING_LIMIT_ENABLED,
    SESSION_COLUMN_IDLE_SEEDING_LIMIT,
    SESSION_COLUMN_SEED_RATIO_LIMITED,
    SESSION_COLUMN_SEED_RATIO_LIMIT,
    SESSION_COLUMN_DOWNLOAD_QUEUE_SIZE,
    SESSION_COLUMN_DOWNLOAD_QUEUE_ENABLED,
    SESSION_COLUMN_SEED_QUEUE_SIZE,
    SESSION_COLUMN_SEED_QUEUE_ENABLED,
    SESSION_COLUMN_INCOMPLETE_DIR,
    SESSION_COLUMN_INCOMPLETE_DIR_ENABLED,
    SESSION_COLUMN_QUEUE_STALLED_MINUTES,
    SESSION_COLUMN_QUEUE_STALLED_ENABLED,
    SESSION_COLUMN_REMOVE_ADDED,
    SESSION_COLUMN_SCRIPT_DONE_ENABLED,
    SESSION_COLUMN_SCRIPT_DONE,
  } from '~helpers/constants/columns';

  let loadingInitial = true;
  let loadingSubmit = false;
  let downloadDir = null;
  let startAddedTorrents = null;
  let renamePartialFiles = null;
  let seedRatioLimited = null;
  let seedRatioLimit = null;
  let idleSeedingLimited = null;
  let idleSeedingLimit = null;
  let downloadQueueSize = null;
  let downloadQueueEnabled = null;
  let seedQueueSize = null;
  let seedQueueEnabled = null;
  let incompleteDir = null;
  let incompleteDirEnabled = null;
  let queueStalledEnabled = null;
  let queueStalledMinutes = null;
  let removeTorrentFile = null;
  let scriptDoneEnabled = null;
  let scriptDone = null;

  session
    .addColumns([
      SESSION_COLUMN_DOWNLOAD_DIR,
      SESSION_COLUMN_START_ADDED,
      SESSION_COLUMN_RENAME_PARTIAL_FILES,
      SESSION_COLUMN_SEED_RATIO_LIMITED,
      SESSION_COLUMN_SEED_RATIO_LIMIT,
      SESSION_COLUMN_IDLE_SEEDING_LIMIT_ENABLED,
      SESSION_COLUMN_IDLE_SEEDING_LIMIT,
      SESSION_COLUMN_DOWNLOAD_QUEUE_SIZE,
      SESSION_COLUMN_DOWNLOAD_QUEUE_ENABLED,
      SESSION_COLUMN_SEED_QUEUE_SIZE,
      SESSION_COLUMN_SEED_QUEUE_ENABLED,
      SESSION_COLUMN_INCOMPLETE_DIR,
      SESSION_COLUMN_INCOMPLETE_DIR_ENABLED,
      SESSION_COLUMN_QUEUE_STALLED_MINUTES,
      SESSION_COLUMN_QUEUE_STALLED_ENABLED,
      SESSION_COLUMN_REMOVE_ADDED,
      SESSION_COLUMN_SCRIPT_DONE_ENABLED,
      SESSION_COLUMN_SCRIPT_DONE,
    ])
    .then(($session) => {
      downloadDir = $session[SESSION_COLUMN_DOWNLOAD_DIR];
      startAddedTorrents = $session[SESSION_COLUMN_START_ADDED];
      renamePartialFiles = $session[SESSION_COLUMN_RENAME_PARTIAL_FILES];
      seedRatioLimited = $session[SESSION_COLUMN_SEED_RATIO_LIMITED];
      seedRatioLimit = $session[SESSION_COLUMN_SEED_RATIO_LIMIT];
      idleSeedingLimited = $session[SESSION_COLUMN_IDLE_SEEDING_LIMIT_ENABLED];
      idleSeedingLimit = $session[SESSION_COLUMN_IDLE_SEEDING_LIMIT];
      downloadQueueSize = $session[SESSION_COLUMN_DOWNLOAD_QUEUE_SIZE];
      downloadQueueEnabled = $session[SESSION_COLUMN_DOWNLOAD_QUEUE_ENABLED];
      seedQueueSize = $session[SESSION_COLUMN_SEED_QUEUE_SIZE];
      seedQueueEnabled = $session[SESSION_COLUMN_SEED_QUEUE_ENABLED];
      incompleteDir = $session[SESSION_COLUMN_INCOMPLETE_DIR];
      incompleteDirEnabled = $session[SESSION_COLUMN_INCOMPLETE_DIR_ENABLED];
      queueStalledMinutes = $session[SESSION_COLUMN_QUEUE_STALLED_MINUTES];
      queueStalledEnabled = $session[SESSION_COLUMN_QUEUE_STALLED_ENABLED];
      removeTorrentFile = $session[SESSION_COLUMN_REMOVE_ADDED];
      scriptDoneEnabled = $session[SESSION_COLUMN_SCRIPT_DONE_ENABLED];
      scriptDone = $session[SESSION_COLUMN_SCRIPT_DONE];
      loadingInitial = false;
    })
    .catch(() => {
      alerts.add(
        '目前无法获取该操作的数据，请稍后重试。',
        'negative'
      );
    });

  const handleSubmit = () => {
    loadingSubmit = true;

    session
      .update({
        [SESSION_COLUMN_DOWNLOAD_DIR]: downloadDir,
        [SESSION_COLUMN_START_ADDED]: startAddedTorrents,
        [SESSION_COLUMN_RENAME_PARTIAL_FILES]: renamePartialFiles,
        [SESSION_COLUMN_SEED_RATIO_LIMITED]: seedRatioLimited,
        [SESSION_COLUMN_SEED_RATIO_LIMIT]: seedRatioLimit,
        [SESSION_COLUMN_IDLE_SEEDING_LIMIT_ENABLED]: idleSeedingLimited,
        [SESSION_COLUMN_IDLE_SEEDING_LIMIT]: idleSeedingLimit,
        [SESSION_COLUMN_DOWNLOAD_QUEUE_SIZE]: downloadQueueSize,
        [SESSION_COLUMN_DOWNLOAD_QUEUE_ENABLED]: downloadQueueEnabled,
        [SESSION_COLUMN_SEED_QUEUE_SIZE]: seedQueueSize,
        [SESSION_COLUMN_SEED_QUEUE_ENABLED]: seedQueueEnabled,
        [SESSION_COLUMN_INCOMPLETE_DIR]: incompleteDir,
        [SESSION_COLUMN_INCOMPLETE_DIR_ENABLED]: incompleteDirEnabled,
        [SESSION_COLUMN_QUEUE_STALLED_MINUTES]: queueStalledMinutes,
        [SESSION_COLUMN_QUEUE_STALLED_ENABLED]: queueStalledEnabled,
        [SESSION_COLUMN_REMOVE_ADDED]: removeTorrentFile,
        [SESSION_COLUMN_SCRIPT_DONE_ENABLED]: scriptDoneEnabled,
        [SESSION_COLUMN_SCRIPT_DONE]: scriptDone,
      })
      .then(() => {
        alerts.add('已成功保存种子设置');
      })
      .catch(() => {
        alerts.add(
          '保存种子设置失败，请重试',
          'negative'
        );
      })
      .finally(() => {
        loadingSubmit = false;
      });
  };
</script>

<div class="wrapper" class:loading-initial="{loadingInitial}">
  <Icon name="SpinnerIcon" />
  <form on:submit|preventDefault="{handleSubmit}">
    <Header text="下载中" />
    <InputPath
      label="下载至"
      bind:value="{downloadDir}"
      hint="这将用作 Transmission 添加新种子时的默认位置。"
    />
    <Checkbox label="添加时自动开始" bind:checked="{startAddedTorrents}" />
    <Checkbox
      label="添加后删除种子文件"
      bind:checked="{removeTorrentFile}"
    />
    <Checkbox
      label="{'将“.part”后缀附加到未完成的文件名称中'}"
      bind:checked="{renamePartialFiles}"
    />
    <Checkbox
      label="未完成目录："
      bind:checked="{incompleteDirEnabled}"
    />
    <InputPath
      bind:value="{incompleteDir}"
      hint="未启用时将默认使用下载目录"
    />
    <Checkbox
      label="完成后运行脚本:"
      bind:checked="{scriptDoneEnabled}"
    />
    <InputPath bind:value="{scriptDone}" />

    <Header text="做种" />
    <Checkbox
      label="按分享率停止做种:"
      bind:checked="{seedRatioLimited}"
    />
    <Input bind:value="{seedRatioLimit}" type="number" step="0.01" />
    <Checkbox
      label="如果空闲时间达到（分钟）则停止做种:"
      bind:checked="{idleSeedingLimited}"
    />
    <Input bind:value="{idleSeedingLimit}" type="number" />

    <Header text="队列" />
    <Checkbox
      label="限制同时下载数:"
      bind:checked="{downloadQueueEnabled}"
    />
    <Input bind:value="{downloadQueueSize}" type="number" />
    <Checkbox
      label="限制同时做种数:"
      bind:checked="{seedQueueEnabled}"
    />
    <Input bind:value="{seedQueueSize}" type="number" />
    <Checkbox
      label="视为不活动需要的时间（分钟）:"
      bind:checked="{queueStalledEnabled}"
    />
    <Input
      bind:value="{queueStalledMinutes}"
      type="number"
      hint="在这段时间内处于非活动状态的 Torrent 将不会被视为同时下载/做种。"
    />

    <div class="buttons">
      <Button type="button" priority="tertiary" on:click="{modals.close}">
        取消
      </Button>
      <Button type="submit" priority="primary" loading="{loadingSubmit}">
        保存设置
      </Button>
    </div>
  </form>
</div>

<style>
  .wrapper {
    min-height: 100%;
    display: flex;
  }

  .wrapper.loading-initial {
    align-items: center;
    justify-content: center;
    fill: var(--color-modal-loading);
  }

  .wrapper > :global(.icon) {
    position: absolute;
    height: 30px;
    width: 30px;
    display: none;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);
  }

  .wrapper.loading-initial > :global(.icon) {
    display: inherit;
  }

  .wrapper.loading-initial form {
    visibility: hidden;
  }

  form {
    width: 100%;
    display: flex;
    flex-direction: column;
    line-height: 1;
    color: var(--color-modal-text);
  }

  form :global(.checkbox) {
    margin-bottom: 15px;
  }

  .buttons {
    flex-grow: 1;
    align-items: flex-end;
    display: flex;
    justify-content: flex-end;
    gap: 10px;
  }
</style>
