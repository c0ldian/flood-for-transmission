<script>
  import Header from './Header.svelte';
  import Button from '~components/Button';
  import InputMultiple from '~components/InputMultiple';
  import Checkbox from '~components/Checkbox';
  import Select from '~components/Select';
  import {
    modals,
    alerts,
    uiColumns,
    paths,
    darkMode,
    switchSpeedColors,
    timeConfig,
    tableHeaderConfig,
    diskUsage,
  } from '~helpers/stores';
  import { orderable } from '~helpers/actions';
  import { PATH_VALIDATION_REGEX } from '~helpers/constants/paths';

  const newColumns = structuredClone($uiColumns);
  let newPaths = [...$paths];
  let newSwitchSpeedColors = $switchSpeedColors;
  let newTimeConfig = $timeConfig;
  let newTableHeaderConfig = $tableHeaderConfig;
  const configuredDarkMode = darkMode.configuredValue;
  let newDarkMode = $configuredDarkMode;
  let newDiskUsage = $diskUsage;

  const darkModeOptions = [
    { label: '自动', value: 'auto' },
    { label: '启用', value: 'enabled' },
    { label: '禁用', value: 'disabled' },
  ];

  const handleSubmit = () => {
    try {
      uiColumns.set(newColumns);
      paths.set(newPaths);
      switchSpeedColors.set(newSwitchSpeedColors);
      timeConfig.set(newTimeConfig);
      tableHeaderConfig.set(newTableHeaderConfig);
      darkMode.set(newDarkMode);
      diskUsage.set(newDiskUsage);
      alerts.add('成功保存用户界面设置');
    } catch (e) {
      console.error(e);
      alerts.add(
        '保存用户界面设置失败，请重试',
        'negative'
      );
    }
  };

  const handleColumnDrop = (sorting) => {
    newColumns.sort((a, b) => sorting.indexOf(a.id) - sorting.indexOf(b.id));
  };
</script>

<form on:submit|preventDefault="{handleSubmit}">
  <Header text="配色方案" />
  <div class="list">
    <Select
      options="{darkModeOptions}"
      on:change="{(event) => (newDarkMode = event.detail)}"
      value="{newDarkMode}"
      direction="below"
      label="深色模式"
    />
  </div>

  <div class="list">
    <Checkbox
      label="改变下载速度颜色"
      hint="这将切换上传和下载的颜色。最初绿色表示下载，蓝色表示上传。"
      bind:checked="{newSwitchSpeedColors}"
    />
  </div>

  <Header text="格式" />
  <div class="list">
    <Checkbox
      label="24小时制"
      hint="如果启用，则使用 24 小时制；如果禁用，则使用 12 小时制。"
      bind:checked="{newTimeConfig}"
    />
  </div>

  <div class="list">
    <Checkbox
      label="标题换行"
      hint="启用后将在标题列中换行"
      bind:checked="{newTableHeaderConfig}"
    />
  </div>

  <Header text="通用路径" />
  <p class="hint">
    这些路径将显示在建议路径后面，您可以在那里选择一条路径。
  </p>
  <InputMultiple
    bind:values="{newPaths}"
    pattern="{PATH_VALIDATION_REGEX}"
    validationMessage="路径必须是绝对路径。"
  />

  <div class="list">
    <Checkbox
      label="显示磁盘使用情况"
      hint="根据通用路径在侧面板显示磁盘使用情况"
      bind:checked="{newDiskUsage}"
    />
  </div>

  <Header text="列名称" />
  <div class="list">
    {#each Object.values(newColumns) as column}
      <div
        class="column"
        draggable="true"
        use:orderable="{handleColumnDrop}"
        id="{column.id}"
      >
        <span>{uiColumns.getColumnLabel(column.id)}</span>
        <Checkbox bind:checked="{column.enabled}" label="启用" />
      </div>
    {/each}
  </div>

  <div class="buttons">
    <Button type="button" priority="tertiary" on:click="{modals.close}">
      取消
    </Button>
    <Button type="submit" priority="primary">保存设置</Button>
  </div>
</form>

<style>
  form {
    min-height: 100%;
    display: flex;
    flex-direction: column;
    line-height: 1;
    color: var(--color-modal-text);
  }

  form :global(.checkbox) :global(.indicator) {
    background: var(--color-checkbox-background-dark);
    border-color: var(--color-checkbox-border-dark);
  }

  .buttons {
    flex-grow: 1;
    align-items: flex-end;
    display: flex;
    justify-content: flex-end;
    gap: 10px;
  }

  .list {
    margin-bottom: 15px;
    border-radius: 3px;
    overflow: hidden;
  }

  .column {
    display: flex;
    justify-content: space-between;
    align-items: center;
    background: var(--color-modal-user-interface-column-background);
    border: 1px solid var(--color-modal-user-interface-column-border);
    cursor: move;
    height: 30px;
    padding: 0 5px;
    font-size: 13px;
  }

  .hint {
    font-size: 12px;
    line-height: 1.5;
    margin-bottom: 8px;
  }
</style>
