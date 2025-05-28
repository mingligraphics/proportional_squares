<script>
    export let currentStep;
    import Scrolly from "../helpers/Scrolly.svelte";

    $: steps = [
    `At several top U.S. universities,
    <span style="background-color:#AC8FBE;padding:3.5px;margin:1px;border-radius:4px;font-weight:500;color:white;">Chinese students</span>
     make up a large share of the <span style="background-color:#E2F2F2;padding:3.5px;margin:1px;border-radius:4px;font-weight:500">international student body</span>. 
     At schools like Cornell and Columbia, nearly half of international students are from China.`,
//     `Note: Data as of May 3. Downloads data are not available for episodes released from Nov. 2015 to Oct. 2017. Episodes within the past 180 days are projected downloads. The Porsche episode does not include YouTube data.
// <br>Source: Acquired's analysis of data from Transistor, Libsyn, Spotify and YouTube`
  ];
</script>
<div class="steps">
    <Scrolly bind:value={currentStep}>
      {#each steps as text, i}
      <div class="step" class:active={currentStep === i}>
        <div class="step-content">
          {@html text}
        </div>
      </div>
      {/each}
    </Scrolly>
  </div>

  <style>
    .step {
    height: 90vh;
    opacity: 0.3;
    transition: opacity 300ms ease;

/* vertically and horizontally center the steps */
    display: flex;
    justify-content: center;
    place-items: center;

}

.step-content {
  position: relative;
  box-sizing: border-box;
  z-index: 1;
  -webkit-transform: translate3d(0, 0, 0); /*for safari z-index issue*/
  height: auto;
  width: 100%;
  max-width: 700px; 
  margin: 0 auto;
  font-family: RetinaLight, sans-serif;
  font-size: 17px;
  line-height: 27px;
  font-weight: 400;
  color: #333;
  padding: 20px 25px 20px;
  box-shadow: 0 0 1px 0 rgba(0, 0, 0, .4);
  overflow: visible;
  background-color: rgba(255, 255, 255, .9);
}

.step.active {
  opacity: 1;
}

.steps {
  position: relative;
  z-index: 2;
  pointer-events: none;
}

/* mark{
  padding-top: 3px;
  padding-bottom: 3px;
  padding-left: 4px;
  padding-right: 4px;
  margin-right: 1px;
  margin-left: 1px;
  border-radius: 4px;
  white-space: nowrap;
  font-weight: 500;
  text-shadow: none;
} */
  </style>