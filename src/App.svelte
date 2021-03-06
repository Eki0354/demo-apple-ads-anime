<script>
  import { onDestroy, onMount } from 'svelte'

  const canvasID = 'canvas-apple-anime'
  const imageSectionID = 'section-images'
  let imageSectionDom = null
  let canvasDom = null

  const imageCount = 176
  const images = Array(imageCount).fill(null)
  const imageWidth = 1458
  const imageHeight = 820

  let isLoading = true
  let canvasHeight = 0
  let canvasWidth = 0
  let imgDX = 0
  let imgDY = 0
  let ctx = null

  let currentScrollTop = 0
  let isScrolling = false

  // 当前帧下标（img标签）
  let currentFrameIndex = -1
  // 目标帧下标（img标签）
  let targetFrameIndex = 0

  // 设定每帧滚动高度，控制滚动速度
  const hpf = 20
  // 设定每帧渲染秒数，控制播放速度（此处测试后使用一般电影帧数）
  const spf = 1 / 23.976

  // 上一帧时间节点
  let lastFramStamp = 0

  const delayTime = [
    20, 20, 20, 20, 20, 20, 15, 15, 15, 15, 15, 15, 10, 10, 10, 10, 10, 10,
  ]

  onMount(async () => {
    await loadImages()

    isLoading = false

    setTimeout(() => {
      // 等待图片加载完成后才监听事件
      initEvent()

      initClient()
    }, 0)
  })

  onDestroy(() => {
    removeEvent()
  })

  function initClient() {
    canvasDom = document.getElementById(canvasID)
    ctx = canvasDom.getContext('2d')

    const { clientWidth, clientHeight } = document.body
    canvasWidth = clientWidth
    canvasHeight = clientHeight

    const scrollTop = (imageCount / 2) * canvasHeight
    imageSectionDom.scrollTo(0, scrollTop)
    currentScrollTop = scrollTop

    const rx = canvasWidth / imageWidth
    const ry = canvasHeight / imageHeight
    canvasDom.style.transform = `scale(${Math.min(rx, ry)})`

    // 高宽至少一屏，防止无法触发滚动事件
    imageSectionDom.style.width = canvasWidth + 'px'
    imageSectionDom.style.height = canvasHeight + 'px'

    play()
  }

  function initEvent() {
    window.addEventListener('resize', initClient)
    imageSectionDom.addEventListener('scroll', handleImagesScroll)
    imageSectionDom.addEventListener('scrollstart', handleImageScrollStart)
    imageSectionDom.addEventListener('scrollstop', handleImageScrollStop)
  }

  function removeEvent() {
    window.removeEventListener('resize', initClient)
    imageSectionDom.removeEventListener('scroll', handleImagesScroll)
    imageSectionDom.removeEventListener('scrollstart', handleImageScrollStart)
    imageSectionDom.removeEventListener('scrollstop', handleImageScrollStop)
  }

  function loadImages() {
    imageSectionDom = document.getElementById(imageSectionID)
    imageSectionDom.innerHTML = ''

    let finishedCount = 0
    const finishCallback = () => finishedCount++
    const baseUrl =
      'https://www.apple.com.cn/105/media/us/airpods-pro/2019/1299e2f5_9206_4470_b28e_08307a42f19b/anim/sequence/large/06-transparency-head/'

    images.forEach((n, i) => {
      const img = document.createElement('img')
      img.className = 'img'
      img.src = `${baseUrl}${i.toString().padStart(4, 0)}.jpg`
      img.draggable = false
      img.onload = img.onerror = finishCallback

      const imgBox = document.createElement('div')
      imgBox.className = 'img-box flex-center'
      imgBox.appendChild(img)

      imageSectionDom.appendChild(imgBox)
      images[i] = img
    })

    const promise = resolve => {
      setTimeout(
        () => (finishedCount < imageCount ? promise(resolve) : resolve()),
        100
      )
    }

    return new Promise(resolve => {
      setTimeout(() => promise(resolve), 2000)
    })
  }

  function play(elapsedTime) {
    let isRenderFrame = true
    if (!elapsedTime || !lastFramStamp) {
      lastFramStamp = elapsedTime
    } else if (elapsedTime - lastFramStamp >= spf) {
      lastFramStamp = elapsedTime - spf
    } else {
      isRenderFrame = false
    }

    const isProgressing = currentFrameIndex !== targetFrameIndex

    if (isProgressing && isRenderFrame) {
      currentFrameIndex < targetFrameIndex
        ? currentFrameIndex++
        : currentFrameIndex--
      ctx.clearRect(0, 0, imageWidth, imageHeight)

      const diff = Math.abs(currentFrameIndex - targetFrameIndex)
      if (currentFrameIndex > -1 && currentFrameIndex < imageCount) {
        if (diff < delayTime) {
          setTimeout(() => {
            ctx.drawImage(images[currentFrameIndex], 0, 0)
          }, delayTime[diff])
        } else {
          ctx.drawImage(images[currentFrameIndex], 0, 0)
        }
      }
    }

    if (isProgressing || isScrolling) requestAnimationFrame(play)
  }

  function handleImagesScroll(e) {
    const diff = e.target.scrollTop - currentScrollTop
    currentScrollTop = e.target.scrollTop

    const diffIndex = Math.round(diff / hpf)
    targetFrameIndex += diffIndex
    play()
  }

  function handleImageScrollStart(e) {
    isScrolling = true
  }

  function handleImageScrollStop(e) {
    isScrolling = false
  }
</script>

<main class="flex-center">
  <section class="section-canvas flex-center">
    {#if isLoading}
      <div class="text-loading">加载中. . .</div>
    {:else}
      <div class="coat">
        <canvas id={canvasID} width={imageWidth} height={imageHeight} />
      </div>
    {/if}
  </section>
  <section id={imageSectionID} class="section-images" />
</main>

<style>
  main {
    height: 100%;
    text-align: center;
    margin: 0 auto;
    background-color: #000;
    color: #fff;
    overflow: hidden;
  }

  .section-canvas {
    width: 100%;
  }

  #canvas-apple-anime {
    transform-origin: 0 50%;
  }

  .section-images {
    position: absolute;
    left: 0;
    right: 0;
    top: 0;
    bottom: 0;
    z-index: 1;
    opacity: 0.001;
    overflow: hidden scroll;
  }

  .section-images::-webkit-scrollbar {
    width: 0;
  }

  :global(.img-box) {
    width: 100%;
    height: 100%;
  }

  :global(.img) {
    width: 100%;
  }

  .text-loading {
    font-size: 20px;
    font-weight: 700;
    padding-top: 4rem;
  }

  .coat {
    width: 100%;
    height: 100%;
  }
</style>
