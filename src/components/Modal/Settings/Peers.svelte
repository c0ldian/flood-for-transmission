<script>
  import Header from './Header.svelte';
  import Checkbox from '~components/Checkbox';
  import Input from '~components/Input';
  import Icon from '~components/Icon';
  import Button from '~components/Button';
  import Select from '~components/Select';
  import { session, modals, alerts } from '~helpers/stores';
  import {
    SESSION_COLUMN_BLOCKLIST_ENABLED,
    SESSION_COLUMN_BLOCKLIST_SIZE,
    SESSION_COLUMN_BLOCKLIST_URL,
    SESSION_COLUMN_DHT_ENABLED,
    SESSION_COLUMN_ENCRYPTION,
    SESSION_COLUMN_LPD_ENABLED,
    SESSION_COLUMN_PEER_LIMIT_GLOBAL,
    SESSION_COLUMN_PEER_LIMIT_PER_TORRENT,
    SESSION_COLUMN_PEX_ENABLED,
  } from '~helpers/constants/columns';

  let loadingInitial = true;
  let submitLoading = false;
  let blocklistUpdateLoading = false;

  let maxPeersPerTorrent = null;
  let maxPeersOverall = null;
  let pexEnabled = null;
  let dhtEnabled = null;
  let lpdEnabled = null;
  let blocklistUrl = null;
  let blocklistEnabled = null;
  let blocklistSize = 0;
  let encryption = null;

  const numberFormatter = new Intl.NumberFormat();
  const encryptionOptions = [
    { label: 'Required', value: 'required' },
    { label: 'Preferred', value: 'preferred' },
    { label: 'Tolerated', value: 'tolerated' },
  ];

  session
    .addColumns([
      SESSION_COLUMN_BLOCKLIST_ENABLED,
      SESSION_COLUMN_BLOCKLIST_SIZE,
      SESSION_COLUMN_BLOCKLIST_URL,
      SESSION_COLUMN_DHT_ENABLED,
      SESSION_COLUMN_ENCRYPTION,
      SESSION_COLUMN_LPD_ENABLED,
      SESSION_COLUMN_PEER_LIMIT_GLOBAL,
      SESSION_COLUMN_PEER_LIMIT_PER_TORRENT,
      SESSION_COLUMN_PEX_ENABLED,
    ])
    .then(($session) => {
      maxPeersPerTorrent = $session[SESSION_COLUMN_PEER_LIMIT_PER_TORRENT];
      maxPeersOverall = $session[SESSION_COLUMN_PEER_LIMIT_GLOBAL];
      pexEnabled = $session[SESSION_COLUMN_PEX_ENABLED];
      dhtEnabled = $session[SESSION_COLUMN_DHT_ENABLED];
      lpdEnabled = $session[SESSION_COLUMN_LPD_ENABLED];
      blocklistUrl = $session[SESSION_COLUMN_BLOCKLIST_URL];
      blocklistEnabled = $session[SESSION_COLUMN_BLOCKLIST_ENABLED];
      blocklistSize = $session[SESSION_COLUMN_BLOCKLIST_SIZE];
      encryption = $session[SESSION_COLUMN_ENCRYPTION];
      loadingInitial = false;
    })
    .catch(() => {
      alerts.add(
        '目前无法获取该操作的数据，请稍后重试。',
        'negative'
      );
    });

  const updateBlocklist = () => {
    blocklistUpdateLoading = true;

    session
      .update({
        'blocklist-url': blocklistUrl,
      })
      .then(session.updateBlocklist)
      .then((newBlocklistSize) => {
        blocklistSize = newBlocklistSize;
        alerts.add('已成功更新您的黑名单');
      })
      .catch(() => {
        alerts.add(
          '更新黑名单失败，请重试',
          'negative'
        );
      })
      .finally(() => {
        blocklistUpdateLoading = false;
      });
  };

  const handleSubmit = () => {
    submitLoading = true;

    session
      .update({
        'peer-limit-per-torrent': maxPeersPerTorrent,
        'peer-limit-global': maxPeersOverall,
        'pex-enabled': pexEnabled,
        'dht-enabled': dhtEnabled,
        'lpd-enabled': lpdEnabled,
        'blocklist-url': blocklistUrl,
        'blocklist-enabled': blocklistEnabled,
        'encryption': encryption,
      })
      .then(() => {
        alerts.add('已成功保存Peers设置');
      })
      .catch(() => {
        alerts.add(
          '保存Peers设置失败，请重试',
          'negative'
        );
      })
      .finally(() => {
        submitLoading = false;
      });
  };
</script>

<div class="wrapper" class:loading-initial="{loadingInitial}">
  <Icon name="SpinnerIcon" />
  <form on:submit|preventDefault="{handleSubmit}">
    <Header text="连接性" />
    <Input
      bind:value="{maxPeersPerTorrent}"
      label="每个 torrent 的最大连接数"
      type="number"
    />
    <Input
      bind:value="{maxPeersOverall}"
      label="全局最大连接数"
      type="number"
    />

    <Header text="设置" />
    <Checkbox bind:checked="{pexEnabled}" label="使用 PEX 查找更多Peers" />
    <Checkbox bind:checked="{dhtEnabled}" label="使用 DHT 查找更多Peers" />
    <Checkbox bind:checked="{lpdEnabled}" label="使用 LPD 查找更多Peers" />

    <Header text="隐私" />
    <Select
      options="{encryptionOptions}"
      on:change="{(event) => (encryption = event.detail)}"
      value="{encryption}"
      direction="below"
      label="加密"
    />

    <Header text="黑名单" />
    <Checkbox bind:checked="{blocklistEnabled}" label="启用黑名单" />
    <Input
      bind:value="{blocklistUrl}"
      type="url"
      hint="黑名单有 {numberFormatter.format(blocklistSize)} 条规则"
    />
    <div class="update-wrapper">
      <Button
        type="button"
        priority="tertiary"
        on:click="{updateBlocklist}"
        loading="{blocklistUpdateLoading}">Update blocklist</Button
      >
    </div>

    <div class="buttons">
      <Button type="button" priority="tertiary" on:click="{modals.close}">
        取消
      </Button>
      <Button type="submit" priority="primary" loading="{submitLoading}">
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

  form :global(.checkbox),
  form :global(.select) {
    margin-bottom: 15px;
  }

  .update-wrapper {
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
