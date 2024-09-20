<script>
  import Header from './Header.svelte';
  import Input from '~components/Input';
  import Checkbox from '~components/Checkbox';
  import Button from '~components/Button';
  import Icon from '~components/Icon';
  import { session, modals, alerts } from '~helpers/stores';
  import {
    SESSION_COLUMN_PEER_PORT,
    SESSION_COLUMN_PEER_PORT_RANDOM_ON_START,
    SESSION_COLUMN_PORT_FORWARDING_ENABLED,
    SESSION_COLUMN_UTP_ENABLED,
  } from '~helpers/constants/columns';

  let loadingInitial = true;
  let submitLoading = false;

  let peerPort = null;
  let randomizePeerPort = null;
  let portForwardingEnabled = null;
  let utpEnabled = null;

  let portOpen = false;

  session.getPortTest().then((isPortOpen) => {
    portOpen = isPortOpen;
  });

  session
    .addColumns([
      SESSION_COLUMN_PEER_PORT,
      SESSION_COLUMN_PEER_PORT_RANDOM_ON_START,
      SESSION_COLUMN_PORT_FORWARDING_ENABLED,
      SESSION_COLUMN_UTP_ENABLED,
    ])
    .then(($session) => {
      peerPort = $session[SESSION_COLUMN_PEER_PORT];
      randomizePeerPort = $session[SESSION_COLUMN_PEER_PORT_RANDOM_ON_START];
      portForwardingEnabled = $session[SESSION_COLUMN_PORT_FORWARDING_ENABLED];
      utpEnabled = $session[SESSION_COLUMN_UTP_ENABLED];
      loadingInitial = false;
    })
    .catch(() => {
      alerts.add(
        '目前无法获取该操作的数据，请稍后重试。',
        'negative'
      );
    });

  const handleSubmit = () => {
    submitLoading = true;

    session
      .update({
        'peer-port': peerPort,
        'peer-port-random-on-start': randomizePeerPort,
        'port-forwarding-enabled': portForwardingEnabled,
        'utp-enabled': utpEnabled,
      })
      .then(() => {
        alerts.add('成功保存网络设置');
      })
      .catch(() => {
        alerts.add(
          '保存网络设置失败，请重试',
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
    <Header text="监听端口" />
    <Input
      label="Peer监听端口"
      bind:value="{peerPort}"
      type="number"
      hint="Port is {portOpen ? 'open' : 'closed'}"
    />
    <Checkbox
      label="启动时随机化端口"
      bind:checked="{randomizePeerPort}"
    />
    <Checkbox
      label="使用路由器的端口转发"
      bind:checked="{portForwardingEnabled}"
    />

    <Header text="设置" />
    <Checkbox
      label="启用 uTP "
      bind:checked="{utpEnabled}"
    />

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
