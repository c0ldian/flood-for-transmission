<script>
  import Header from './Header.svelte';
  import Checkbox from '~components/Checkbox';
  import Input from '~components/Input';
  import Button from '~components/Button';
  import Icon from '~components/Icon';
  import { session, modals, alerts } from '~helpers/stores';
  import { minutesToTime, timeToMinutes } from '~helpers/timeHelper';
  import {
    SESSION_COLUMN_SPEED_LIMIT_UP_ENABLED,
    SESSION_COLUMN_SPEED_LIMIT_UP,
    SESSION_COLUMN_SPEED_LIMIT_DOWN_ENABLED,
    SESSION_COLUMN_SPEED_LIMIT_DOWN,
    SESSION_COLUMN_ALT_SPEED_ENABLED,
    SESSION_COLUMN_ALT_SPEED_UP,
    SESSION_COLUMN_ALT_SPEED_DOWN,
    SESSION_COLUMN_ALT_SPEED_TIME_ENABLED,
    SESSION_COLUMN_ALT_SPEED_TIME_BEGIN,
    SESSION_COLUMN_ALT_SPEED_TIME_END,
  } from '~helpers/constants/columns';

  let loadingInitial = true;
  let loadingSubmit = false;

  let uploadLimitEnabled = null;
  let uploadLimit = null;
  let downloadLimitEnabled = null;
  let downloadLimit = null;
  let altLimitsEnabled = null;
  let altUploadLimit = null;
  let altDownloadLimit = null;
  let altSpeedTimeEnabled = null;
  let altSpeedTimeBegin = null;
  let altSpeedTimeEnd = null;

  session
    .addColumns([
      SESSION_COLUMN_SPEED_LIMIT_UP_ENABLED,
      SESSION_COLUMN_SPEED_LIMIT_UP,
      SESSION_COLUMN_SPEED_LIMIT_DOWN_ENABLED,
      SESSION_COLUMN_SPEED_LIMIT_DOWN,
      SESSION_COLUMN_ALT_SPEED_ENABLED,
      SESSION_COLUMN_ALT_SPEED_UP,
      SESSION_COLUMN_ALT_SPEED_DOWN,
      SESSION_COLUMN_ALT_SPEED_TIME_ENABLED,
      SESSION_COLUMN_ALT_SPEED_TIME_BEGIN,
      SESSION_COLUMN_ALT_SPEED_TIME_END,
    ])
    .then(($session) => {
      uploadLimitEnabled = $session[SESSION_COLUMN_SPEED_LIMIT_UP_ENABLED];
      uploadLimit = $session[SESSION_COLUMN_SPEED_LIMIT_UP];
      downloadLimitEnabled = $session[SESSION_COLUMN_SPEED_LIMIT_DOWN_ENABLED];
      downloadLimit = $session[SESSION_COLUMN_SPEED_LIMIT_DOWN];
      altLimitsEnabled = $session[SESSION_COLUMN_ALT_SPEED_ENABLED];
      altUploadLimit = $session[SESSION_COLUMN_ALT_SPEED_UP];
      altDownloadLimit = $session[SESSION_COLUMN_ALT_SPEED_DOWN];
      altSpeedTimeEnabled = $session[SESSION_COLUMN_ALT_SPEED_TIME_ENABLED];
      altSpeedTimeBegin = minutesToTime(
        $session[SESSION_COLUMN_ALT_SPEED_TIME_BEGIN]
      );
      altSpeedTimeEnd = minutesToTime(
        $session[SESSION_COLUMN_ALT_SPEED_TIME_END]
      );

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
        'speed-limit-up-enabled': uploadLimitEnabled,
        'speed-limit-up': uploadLimit,
        'speed-limit-down-enabled': downloadLimitEnabled,
        'speed-limit-down': downloadLimit,
        'alt-speed-enabled': altLimitsEnabled,
        'alt-speed-up': altUploadLimit,
        'alt-speed-down': altDownloadLimit,
        'alt-speed-time-enabled': altSpeedTimeEnabled,
        'alt-speed-time-begin': timeToMinutes(altSpeedTimeBegin),
        'alt-speed-time-end': timeToMinutes(altSpeedTimeEnd),
      })
      .then(() => {
        alerts.add('成功保存速度设置');
      })
      .catch(() => {
        alerts.add(
          '保存速度设置失败，请重试',
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
    <Header text="限速" />
    <Checkbox bind:checked="{uploadLimitEnabled}" label="上传 (kB/s)" />
    <Input bind:value="{uploadLimit}" type="number" />

    <Checkbox bind:checked="{downloadLimitEnabled}" label="下载 (kB/s)" />
    <Input bind:value="{downloadLimit}" type="number" />

    <Header text="备用限速" />
    <Checkbox
      bind:checked="{altLimitsEnabled}"
      label="已启用备用限速"
    />

    <Input bind:value="{altUploadLimit}" label="上传 (kB/s)" type="number" />

    <Input
      bind:value="{altDownloadLimit}"
      label="下载 (kB/s)"
      type="number"
    />

    <Checkbox bind:checked="{altSpeedTimeEnabled}" label="计划时间" />

    <Input bind:value="{altSpeedTimeBegin}" label="从" type="time" />
    <Input bind:value="{altSpeedTimeEnd}" label="至" type="time" />

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
