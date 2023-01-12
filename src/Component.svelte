<script>
  import { getContext, onMount } from "svelte"

  export let autoStepTime = 3
  export let showStepper = true;
  export let stepperHighlight;
  export let stepperBackColor;

  const { styleable } = getContext("sdk")
  const component = getContext("component")

  let container
  let currentChild = -1;
  let childCount = 0;
  let intervalId;

  component.subscribe(c => {
    childCount = c.children
  })

  const stepperStyle = `background-color: ${stepperBackColor}; border-color: ${stepperHighlight}`
  const selectedStepperStyle = `background-color: ${stepperHighlight}; border-color: ${stepperHighlight}`

  $: stepperVisible = showStepper && childCount > 1;

  function cycleChild(index=-1) {
    if (!container || childCount < 2) return

    const lastChild = childCount - 1

    if (index > -1) {
      currentChild = index;
    } else if (currentChild === lastChild) {
      currentChild = 0;
    } else {
      currentChild = currentChild + 1
    }

    for (let counter = 0; counter < childCount; counter++) {
      const child = container.children[counter]
      if (counter === currentChild) {
        child.style.setProperty('display', '');
      } else {
        child.style.setProperty('display', 'none', 'important');
      }
    }     

  }

  function startStepping(index=0) {
    if (intervalId) clearInterval(intervalId)
    cycleChild(index)
    intervalId = setInterval(cycleChild, autoStepTime * 1000);
  }

  onMount ( () =>  { 
    startStepping()
  })

</script>

<div use:styleable={$component.styles}>
  <div id="container" bind:this={container}>
    <slot/>
    {#if stepperVisible} 
      <div id="stepper-container">
        {#each [...Array(childCount).keys()] as index}
          <div 
            class="stepper" 
            style={index === currentChild ? selectedStepperStyle : stepperStyle}
            on:click={() => startStepping(index)}></div>          
        {/each}
      </div>
    {/if}
    
  </div>
</div>

<style>

  #container {
    position: relative;
  }

  #container > * {
    position: absolute;
    top:0;
    left:0;
    width: 100%;
  }

  #stepper-container {
    position: absolute;
    bottom:0;
    right: 0;
    z-index: 99;
    padding: 5px;
    width: auto;
    height: 35px;
    display: block;
    text-align: end;
    margin-top: auto;
  }

  .stepper {
    border-radius: 50%;
    width:7px;
    height:7px;
    border: 1px solid #000;
    background-color: white;
    display: inline-block;
    margin: 5px;
    cursor: pointer;
  }

  .selectedStepper {
    background-color: black;
  }

</style>
