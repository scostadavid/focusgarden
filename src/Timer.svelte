<script lang="ts">
  import { onMount, afterUpdate } from 'svelte';
  import SfxController from './SFXController.svelte';

  enum TimerState {
    INITIALIZED,
    RUNNING,
    PAUSED
  }

  enum ModeState {
    FOCUS,
    REST
  }

  const SHORT_BREAK_MINUTES: number = 5;
  const LONG_BREAK_MINUTES: number = 15;
  const POMODORO_MINUTES: number = import.meta.env.DEV ? 2 : 25;

  let timerId: NodeJS.Timer = null;
  let timerCurrentMode: ModeState = ModeState.FOCUS;
  let timerTimelabel: string = `${POMODORO_MINUTES.toString()}:00`;
  let timerTargetSeconds: number = null;
  let timerInitalTargetMinutes: number = POMODORO_MINUTES;
  let timerState: TimerState = TimerState.INITIALIZED;
  let activeTab: ModeState = ModeState.FOCUS; 

  const modeLabel = {
    [ModeState.FOCUS]: 'Focus',
    [ModeState.REST]: 'Rest'
  };

  const triggerAudio = new Audio(import.meta.env.DEV ? '/notification.wav' : './notification.wav');

  const setup = (minutes: number, mode: ModeState): void => {
    clearInterval(timerId);
    timerTimelabel = `${minutes.toString().padStart(2, '0')}:00`;
    timerState = TimerState.INITIALIZED;
    timerCurrentMode = mode;
    timerTargetSeconds = minutes * 60;
    timerInitalTargetMinutes = minutes;
    activeTab = mode; 
  };

  const reset = (minutes: number): void => {
    setup(minutes, timerCurrentMode);
  };

  const triggerAction = (): void => {
    switch (timerState) {
      case TimerState.INITIALIZED:
        timerId = setInterval(_tick, 1000);
        triggerAudio.play();
        timerState = TimerState.RUNNING;
        break;
      case TimerState.RUNNING:
        timerState = TimerState.PAUSED;
        break;
      case TimerState.PAUSED:
        triggerAudio.play();
        timerState = TimerState.RUNNING;
        break;
    }
  };

  function switchMode(): void {
    triggerAudio.play();
    if (timerCurrentMode === ModeState.FOCUS) {
      timerCurrentMode = ModeState.REST;
      setup(SHORT_BREAK_MINUTES, timerCurrentMode);
      document.title = `(${timerTimelabel}) Focus Garden | ${modeLabel[timerCurrentMode]}`;
      triggerAction();
      return;
    }
    timerCurrentMode = ModeState.FOCUS;
    document.title = `(${timerTimelabel}) Focus Garden | ${modeLabel[timerCurrentMode]}`;
    setup(POMODORO_MINUTES, timerCurrentMode);
    return;
  };

  function _tick(): void {
    if (timerTargetSeconds <= 0) {
      switchMode();
      return;
    }
    if (timerState === TimerState.PAUSED) return;
    timerTargetSeconds--;
    const minutes = Math.floor(timerTargetSeconds / 60);
    const seconds = timerTargetSeconds - minutes * 60;
    timerTimelabel = `${minutes.toString().padStart(2, '0')}:${seconds.toString().padStart(2, '0')}`
  };

  onMount(() => {
    document.title = `(${timerTimelabel}) Focus Garden | ${modeLabel[timerCurrentMode]}`;
    setup(POMODORO_MINUTES, timerCurrentMode);
  });

  afterUpdate(() => {
    document.title = `(${timerTimelabel}) Focus Garden | ${modeLabel[timerCurrentMode]}`
  });

  const setFocusTime = () => setup(POMODORO_MINUTES, ModeState.FOCUS);
  const setShortBreak = () => setup(SHORT_BREAK_MINUTES, ModeState.REST);
  const setLongBreak = () => setup(LONG_BREAK_MINUTES, ModeState.REST);
</script>

<section class="w-full flex justify-center my-4">
  <div class="card clock my-4 rounded-xl">

    <div role="tablist" class="tabs tabs-boxed m-4 bg-base-300">
      <button role="tab" class="tab {activeTab === ModeState.FOCUS ? 'tab-active' : ''}" on:click={setFocusTime}>Focus time</button>
      <button role="tab" class="tab {activeTab === ModeState.REST && timerInitalTargetMinutes === SHORT_BREAK_MINUTES ? 'tab-active' : ''}" on:click={setShortBreak}>Short break</button>
      <button role="tab" class="tab {activeTab === ModeState.REST && timerInitalTargetMinutes === LONG_BREAK_MINUTES ? 'tab-active' : ''}" on:click={setLongBreak}>Long break</button>
    </div>

    <div class="flex flex-col items-center">
      <p class="text-[3em] font-bold">{timerTimelabel}</p>
      <p class="text-[1.2rem]">It's {modeLabel[timerCurrentMode]} time</p>
    </div>

    <div class="clock--options m-4">
      <button class="btn btn-primary" on:click={triggerAction}>
        {#if timerState === TimerState.RUNNING}
          <svg xmlns="http://www.w3.org/2000/svg" height="1em" viewBox="0 0 320 512"><path d="M48 64C21.5 64 0 85.5 0 112V400c0 26.5 21.5 48 48 48H80c26.5 0 48-21.5 48-48V112c0-26.5-21.5-48-48-48H48zm192 0c-26.5 0-48 21.5-48 48V400c0 26.5 21.5 48 48 48h32c26.5 0 48-21.5 48-48V112c0-26.5-21.5-48-48-48H240z"/></svg>
        {:else}
          <svg xmlns="http://www.w3.org/2000/svg" height="1em" viewBox="0 0 384 512"><path d="M73 39c-14.8-9.1-33.4-9.4-48.5-.9S0 62.6 0 80V432c0 17.4 9.4 33.4 24.5 41.9s33.7 8.1 48.5-.9L361 297c14.3-8.7 23-24.2 23-41s-8.7-32.2-23-41L73 39z"/></svg>
        {/if}
      </button>
      <button class="btn btn-primary" on:click={() => reset(timerInitalTargetMinutes)}>
        <svg xmlns="http://www.w3.org/2000/svg" height="1em" viewBox="0 0 512 512"><path d="M463.5 224H472c13.3 0 24-10.7 24-24V72c0-9.7-5.8-18.5-14.8-22.2s-19.3-1.7-26.2 5.2L413.4 96.6c-87.6-86.5-228.7-86.2-315.8 1c-87.5 87.5-87.5 229.3 0 316.8s229.3 87.5 316.8 0c12.5-12.5 12.5-32.8 0-45.3s-32.8-12.5-45.3 0c-62.5 62.5-163.8 62.5-226.3 0s-62.5-163.8 0-226.3c62.2-62.2 162.7-62.5 225.3-1L327 183c-6.9 6.9-8.9 17.2-5.2 26.2s12.5 14.8 22.2 14.8H463.5z"/></svg>
      </button>
    </div>

    <section class="sfx w-full flex flex-col justify-center p-4">
      <SfxController icon={'fire'} sfx={'fire'} />
      <SfxController icon={'rain'} sfx={'rain'} />
      <SfxController icon={'coffe'} sfx={'caffe'} />
    </section>
  </div>
</section>

<style>
  .clock {
    width: 500px;
  }

  .clock--options {
    display: grid;
    grid-template-columns: repeat(2, 1fr);
    grid-gap: .5rem;
  }

  @media (max-width: 700px) {
    .clock {
      width: 350px;
    }
  }
</style>
