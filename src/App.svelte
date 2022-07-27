<script lang="ts">
  import { onMount, tick } from "svelte";

  let activeIndex = 0;
  let activeBarWidth = 0;
  let activeBarLeft = 0;
  const tabItems: HTMLDivElement[] = Array.from({ length: 10 }, () => null);
  let navThis: HTMLDivElement = null;
  let activeBarThis: HTMLDivElement = null;
  let middleScreen = window.innerWidth / 2;

  const syncActiveBar = () => {
    const { offsetLeft, offsetWidth } = tabItems[activeIndex] ?? {};
    activeBarLeft = offsetLeft;
    activeBarWidth = offsetWidth;
  };

  const syncNavScrolling = () => {
    const { x, width } = activeBarThis.getBoundingClientRect();
    const { scrollLeft } = navThis;
    const { innerWidth } = window;
    if (x < 0) {
      navThis.scroll({
        left: scrollLeft + x,
        behavior: "smooth",
      });
      return;
    }
    if (x > innerWidth - width) {
      navThis.scroll({
        left: scrollLeft + width,
        behavior: "smooth",
      });
      return;
    }
  };

  const syncActiveBarAndScrolling = () => {
    syncActiveBar();
    clearTimeout(timer);
    timer = setTimeout(() => {
      syncNavScrolling();
    }, 500);
  };

  let timer = null;

  onMount(() => {
    syncActiveBar();
  });

  let inTouching = false;

  let pageTouchStartX = 0;

  let pageWrapperPageOffsetX = 0;

  let pageWrapperPageX = 0;

  $: pageWrapperPageLeftStyle = `-${
    pageWrapperPageOffsetX * 1.2 + pageWrapperPageX
  }px`;

  const pageTouchStart = (e: TouchEvent) => {
    const {
      touches: [{ pageX }],
    } = e;
    pageTouchStartX = pageX;
    inTouching = true;
  };

  const pageTouchMove = (e: TouchEvent) => {
    if (inTouching) {
      const {
        touches: [{ pageX }],
      } = e;
      pageWrapperPageOffsetX = pageTouchStartX - pageX;
      let p = pageWrapperPageOffsetX / middleScreen;
      p = p > 1 ? 1 : p;
      p = p < -1 ? -1 : p;
      if (pageWrapperPageOffsetX > 0) {
        if (activeIndex + 1 > 9) {
          return;
        }
        const nextItem = tabItems[activeIndex + 1];
        const item = tabItems[activeIndex];
        const offsetLeft = nextItem.offsetLeft - item.offsetLeft;
        const width = nextItem.offsetWidth - item.offsetWidth;
        activeBarLeft = item.offsetLeft + offsetLeft * p;
        activeBarWidth = item.offsetWidth + width * p;
      } else if (pageWrapperPageOffsetX < 0) {
        if (activeIndex - 1 < 0) {
          return;
        }
        const lastItem = tabItems[activeIndex - 1];
        const item = tabItems[activeIndex];
        const offsetLeft = item.offsetLeft - lastItem.offsetLeft;
        const width = item.offsetWidth - lastItem.offsetWidth;
        activeBarLeft = item.offsetLeft + offsetLeft * p;
        activeBarWidth = item.offsetWidth + width * p;
      }
    }
  };

  const pageTouchEnd = (e: TouchEvent) => {
    const {
      changedTouches: [{ pageX }],
    } = e;
    inTouching = false;
    pageWrapperPageOffsetX = pageTouchStartX - pageX;

    const finalOffsetX = pageTouchStartX - pageX;
    console.log(finalOffsetX, middleScreen);
    if (Math.abs(finalOffsetX) >= middleScreen) {
      if (finalOffsetX > 0) {
        if (activeIndex + 1 > 9) {
          pageWrapperPageOffsetX = 0;
          syncActiveBarAndScrolling();
          return;
        }
        activeIndex += 1;
        pageWrapperPageX += window.innerWidth;
      } else {
        if (activeIndex - 1 < 0) {
          pageWrapperPageOffsetX = 0;
          syncActiveBarAndScrolling();
          return;
        }
        activeIndex -= 1;
        pageWrapperPageX -= window.innerWidth;
      }
      syncActiveBarAndScrolling();
    }
    syncActiveBarAndScrolling();
    pageWrapperPageOffsetX = 0;
  };
</script>

<div class="screen">
  <div class="nav" bind:this={navThis}>
    <div class="wrapper">
      {#each Array(10) as _, i (i)}
        <div
          class="item"
          bind:this={tabItems[i]}
          class:active={i === activeIndex}
          on:click={() => {
            activeIndex = i;
            pageWrapperPageX = i * window.innerWidth;
            syncActiveBarAndScrolling();
          }}
        >
          tab-{Array.from(
            { length: Math.floor(Math.random() * 3) + 1 },
            () => 0
          ).join("")}
        </div>
      {/each}
    </div>
    <div
      class="active-bar"
      style="left:{activeBarLeft}px;width:{activeBarWidth}px;"
      bind:this={activeBarThis}
      class:transition={!inTouching}
    />
  </div>
  <div
    class="pages"
    on:touchstart={pageTouchStart}
    on:touchmove={pageTouchMove}
    on:touchend={pageTouchEnd}
  >
    <div
      class="wrapper"
      style="left:{pageWrapperPageLeftStyle};"
      class:transition={!inTouching}
    >
      {#each Array(10) as _, i (i)}
        <div class="page">
          <div class="text">Page-{i}</div>
        </div>
      {/each}
    </div>
  </div>
</div>

<style lang="scss">
  :global(body, html) {
    overscroll-behavior: none;
  }
  :global(body) {
    margin: 0;
    padding: 0;
    background-color: #eee;
    -webkit-tap-highlight-color: rgba($color: #000000, $alpha: 0);
  }
  .screen {
    max-width: 750px;
    height: 100vh;
    margin: 0 auto;
    overflow: hidden;
    background-color: rgb(250, 250, 250);
    display: flex;
    flex-flow: column nowrap;
    .nav {
      $padding-bottom: 2px;
      overflow-x: auto;
      position: relative;
      flex-shrink: 0;
      background-color: #fff;
      padding-bottom: $padding-bottom;
      border: #ddd solid 1px;
      &::-webkit-scrollbar {
        display: none;
      }
      .active-bar {
        width: 54px;
        height: 2px;
        background-color: rgb(231, 77, 149);
        border-radius: 2px;
        position: absolute;
        bottom: $padding-bottom;
        left: 0;
        &.transition {
          transition: all 0.2s;
        }
      }
      .wrapper {
        display: flex;
        align-items: center;
        margin: 0 10px;
        gap: 20px;
        .item {
          cursor: pointer;
          height: 40px;
          display: flex;
          align-items: center;
          justify-content: center;
          font-size: 16px;
          white-space: nowrap;
          padding: 0 6px;
          color: #666;
          &.active {
            color: rgb(231, 77, 149);
          }
        }
      }
    }
    .pages {
      flex: 1;
      overflow: hidden;
      position: relative;
      .wrapper {
        position: absolute;
        top: 0;
        left: 0;
        display: flex;
        height: 100%;
        will-change: left;
        &.transition {
          transition: left 0.2s;
        }
        .page {
          flex-shrink: 0;
          width: 100vw;
          max-width: 750px;
          height: 100%;
          display: flex;
          align-items: center;
          justify-content: center;
          .text {
            font-size: 50px;
            color: #aaa;
          }
        }
      }
    }
  }
</style>
