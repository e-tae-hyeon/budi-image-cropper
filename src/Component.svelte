<script>
  import { getContext, onDestroy } from "svelte"

  export let field;
  export let validation;
  export let label
  export let file;
  export let ratio;
  export let modal = false;
  export let endpoint;
  export let url;

  export let fieldApi;
  export let fieldState;
  export let fieldSchema;
  
  const { styleable } = getContext("sdk")
  const component = getContext("component")
  const formContext = getContext('form')
  const formStep = getContext('form-step')

  const formApi = formContext?.formApi
  $: formField = formApi?.registerField(field, 'number', undefined, false, validation, formStep)

  $: unsubscribe = formField?.subscribe((value) => {
    fieldState = value?.fieldState;
    fieldApi = value?.fieldApi;
    fieldSchema = value?.fieldSchema
    console.log(value)
  })
  
  let img
  let input;
  let files;
  let cropper;

  $: if (files) {
    file = files[0]
    url = URL.createObjectURL(file)
    modal = true
  }

  $: if (img) {
    img.addEventListener('load', initCropper);
  }

  const nav = document.querySelector('.nav-wrapper')

  $: if (modal) {
    nav.style.zIndex = 0
  } else {
    nav.style.zIndex = ''
  }

  const onResizeWindow = () => {
    const nav = document.querySelector('.nav-wrapper')
    setTimeout(() => {
      if (modal) {
        nav.style.zIndex = 0
      } else {
        nav.style.zIndex = ''
      }
    }, 0)
  }
	
	const initCropper = () => {
    if (cropper) {
      cropper.destroy()
    }
		cropper = new Cropper(img, {
      viewMode: 1,
			aspectRatio: eval(ratio),
      zoomable: false,
      guides: false,
		});		
	}

  const openFileSelect = () => {
    input.click()
  }

  const onRemove = () => {
    files = undefined
    file = undefined
    url = undefined
    fieldApi.setValue(null)
  }

  const onSave = async () => {
    const crop = cropper.getData()
    const formData = new FormData()

    let blob = file

    if (!blob) {
      blob = await fetch(url).then(res => res.blob())
    }

    formData.append('images', blob)
    
    formData.append('crops', JSON.stringify([{ x: crop.x, y: crop.y, width: crop.width, height: crop.height }]))
    const response = await fetch(endpoint, {
      method: 'POST',
      body: formData
    })
    
    const data = await response.json()
    
    url = data[0].path
    fieldApi.setValue(data[0].id)
    modal = false
  }

  onDestroy(() => {
    fieldApi?.deregister();
    unsubscribe?.()
  })
</script>

<svelte:head>
	<link  href="https://cdnjs.cloudflare.com/ajax/libs/cropperjs/2.0.0-alpha.2/cropper.css" rel="stylesheet">
	<script src="https://cdnjs.cloudflare.com/ajax/libs/cropperjs/2.0.0-alpha.2/cropper.min.js"></script>
</svelte:head>

<svelte:window
  on:resize={onResizeWindow}
/>

{#if !formContext}
<div class="placeholder">Form components need to be wrapped in a form</div>
{:else}
<div use:styleable={$component.styles} class="spectrum-Form-Item">
  <label class="spectrum-FieldLabel spectrum-Form-itemLabel spectrum-FieldLabel--left spectrum-FieldLabel--sizeM">{label}</label>
  <div class="spectrum-Form-itemField">
    {#if !url || !fieldState.value}
      <div class="spectrum-Dropzone">
        <div class="spectrum-IllustratedMessage spectrumIllustratedMessage--cta">
          <input type="file" hidden accept="image/*" bind:this={input} bind:files />
          <svg class="spectrum-IllustratedMessage-illustration" width="125" height="60" viewBox="0 0 199 97.7"><defs><style>.cls-1,
            .cls-2 {
              fill: none;
              stroke-linecap: round;
              stroke-linejoin: round;
            }
            .cls-1 {
              stroke-width: 3px;
            }
            .cls-2 {
              stroke-width: 2px;
            }
          </style></defs><path class="cls-1" d="M110.53,85.66,100.26,95.89a1.09,1.09,0,0,1-1.52,0L88.47,85.66"></path><line class="cls-1" x1="99.5" y1="95.5" x2="99.5" y2="58.5"></line><path class="cls-1" d="M105.5,73.5h19a2,2,0,0,0,2-2v-43"></path><path class="cls-1" d="M126.5,22.5h-19a2,2,0,0,1-2-2V1.5h-31a2,2,0,0,0-2,2v68a2,2,0,0,0,2,2h19"></path><line class="cls-1" x1="105.5" y1="1.5" x2="126.5" y2="22.5"></line><path class="cls-2" d="M47.93,50.49a5,5,0,1,0-4.83-5A4.93,4.93,0,0,0,47.93,50.49Z"></path><path class="cls-2" d="M36.6,65.93,42.05,60A2.06,2.06,0,0,1,45,60l12.68,13.2"></path><path class="cls-2" d="M3.14,73.23,22.42,53.76a1.65,1.65,0,0,1,2.38,0l19.05,19.7"></path><path class="cls-1" d="M139.5,36.5H196A1.49,1.49,0,0,1,197.5,38V72A1.49,1.49,0,0,1,196,73.5H141A1.49,1.49,0,0,1,139.5,72V32A1.49,1.49,0,0,1,141,30.5H154a2.43,2.43,0,0,1,1.67.66l6,5.66"></path><rect class="cls-1" x="1.5" y="34.5" width="58" height="39" rx="2" ry="2"></rect></svg>
          <h2 class="spectrum-Heading spectrum-Heading--sizeL spectrum-Heading--light spectrum-IllustratedMessage-heading svelte-1u4iys9">이미지 업로드</h2>
          <p class="spectrum-Body spectrum-Body--sizeS spectrum-IllustratedMessage-description svelte-1u4iys9"><label for="c91927045464a4f63b940b0678f3122d4" class="spectrum-Link" on:click={openFileSelect}>파일 선택하기</label></p>
        </div>
      </div>
    {:else}
      <div class="gallery">
        <div style="display: flex; justify-content: flex-end; align-items: center;">
          <div style="display: flex; gap: 8px;">
            <button class="spectrum-Button spectrum-Button--sizeM spectrum-Button--cta" contenteditable="false" draggable="false" on:click={() => { modal = true }}>편집</button>
            <button class="spectrum-Button spectrum-Button--outline spectrum-Button--negative spectrum-Button--sizeM spectrum-Button--iconOnly" aria-label="Delete" on:click={onRemove}>
              <svg class="spectrum-Icon spectrum-Icon--sizeM" focusable="false" aria-hidden="true">
                <use xlink:href="#spectrum-icon-18-Delete" />
              </svg>
            </button>          
          </div>
        </div>

        <a href={url} style="max-width: 400px; overflow: hidden; text-overflow: ellipsis;">{url}</a>

        <div class="image">
          <img src={url} draggable="false" alt="preview" crossorigin="anonymous" />
        </div>
      </div>
    {/if}
  </div>
</div>

  {#if url}
    <div class={`spectrum-Modal-wrapper ${modal ? 'is-open' : ''}`} style="background-color: rgba(0,0,0,0.5); z-index: 10000000; padding: 60px;">
      <div class={`spectrum-Modal ${modal ? 'is-open' : ''}`} data-testid="modal" style="">
        <section class="spectrum-Dialog spectrum-Dialog--fullscreen" role="alertdialog" tabindex="-1" aria-modal="true">
          <div class="spectrum-Dialog-grid">
            <section class="spectrum-Dialog-content" style="padding-top: 12px;">
              <div>
                {#if modal}
                  <img bind:this={img} src={url} alt="" crossorigin="anonymous">
                {/if}
              </div>
            </section>
            <div class="spectrum-ButtonGroup spectrum-Dialog-buttonGroup spectrum-Dialog-buttonGroup--noFooter">
              <button class="spectrum-Button spectrum-Button--sizeM spectrum-Button--outline spectrum-Button--secondary spectrum-ButtonGroup-item" on:click={() => { modal = false }} type="button">
                <span class="spectrum-Button-label">닫기</span>
              </button>
              <button class="spectrum-Button spectrum-Button--sizeM spectrum-Button--fill spectrum-Button--cta spectrum-ButtonGroup-item" type="button" on:click={onSave}>
                <span class="spectrum-Button-label">저장</span>
              </button>
            </div>
          </div>
        </section>
      </div>
    </div>
  {/if}
{/if}


<style>
  .gallery {
    display: flex;
    flex-direction: column;
    justify-content: flex-start;
    align-items: stretch;
    background-color: var(--spectrum-global-color-gray-50);
    color: var(--spectrum-alias-text-color);
    font-size: var(--spectrum-alias-item-text-size-m);
    box-sizing: border-box;
    border: var(--spectrum-alias-border-size-thin) var(--spectrum-alias-border-color) solid;
    border-radius: var(--spectrum-alias-border-radius-regular);
    padding: 10px;
    margin-bottom: 10px;
    position: relative;
    gap: 16px;
  }

  .image {
    max-width: 400px;
    max-height: 400px;
    width: 100%;
    display: flex;
    align-self: center;
  }

  .image img {
    object-fit: contain;
    width: 100%;
  }
</style>

