<script>
  import data from "./data/schools.json";
  import { scaleSqrt } from "d3-scale";
  import { extent, max, group } from 'd3-array';
  import { tweened } from 'svelte/motion';
  import { cubicOut } from "svelte/easing";
  import { interpolate } from "d3-interpolate";
  import { fly, fade, scale } from 'svelte/transition';
  import Steps from "/components/Steps.svelte";
  let currentStep;

  console.log(data);

  let width = 700;
  let height = 530;

  let background_Color = "#CED3DD";//#CED3DD #C6E8EB
  let share_Color = "#683C8F";
  
  const margin = {
      top:15,
      right: 10,
      bottom: 15,
      left: 50
    };

  $: innerWidth = width - margin.left - margin.right;
  let innerHeight = height - margin.top - margin.bottom;

  $: students_range = extent(data, d => d.intl_students);

  $: rScale = scaleSqrt().domain(students_range).range(width > 590 ? [17, 206] : width > 470 ? [12, 160] : width > 370 ? [8, 120] : [6, 100]);

  let initialData = data.sort((a, b) => a.ranking - b.ranking);
  let renderedData = initialData.map((d,i) => ({
  ...d,
  group: i < 4 ? "A" : i < 7 ? "B" : "C"
}));

$: row_width = 30;
$: gap = width/20;

const grouped = group(renderedData, d => d.group);

//get the biggest values of intl_students for each group and get their widths
$: max_A = rScale(max(grouped.get("A"), d => d.intl_students));
$: max_B = rScale(max(grouped.get("B"), d => d.intl_students));
$: max_C = rScale(max(grouped.get("C"), d => d.intl_students));

// Make a Set of objects with their assigned column label
const columnAssignments = new Map();

["A", "B", "C"].forEach(groupKey => {
  const items = grouped.get(groupKey) || [];
  for (let i = 0; i < 4; i++) {
    const item = items[i];
    if (item) {
      columnAssignments.set(item, `Column_${i + 1}`);
    }
  }
});

// Create new data with "column" field
const newData = renderedData.map(d => ({
  ...d,
  column: columnAssignments.get(d) || null
}));

$: max_Column1 = rScale(max(newData.filter(d => d.column === "Column_1"), d => d.intl_students));

const lookbackSteps = 3;

$: processedData = newData.map((d, i, arr) => {
  const result = { ...d };

  for (let step = 1; step <= lookbackSteps; step++) {
    const prev = arr[i - step];
    result[`prev${step}Intl`] = prev?.intl_students ?? null;
  }

  return result;
});

$: newData_before = processedData.map(item => ({
  ...item,
  case: "before",
  share_china: 0  // add the new column
}));

$: newData_after = processedData.map(item => ({
  ...item,
  case: "after"   // add the new column
}));

$: console.log(tweenedPoints)

$: tweenedPoints = tweened(newData_before, {
      delay: 1000,
      duration: 3000,
      easing: cubicOut,
      interpolate: (a, b) => interpolate(a, b)
    });

$:  tweenedPoints.set(newData_after);

const prev2Intl_value = 2318;
const prev1Intl_value = 4073;
const annotation_value = 5627;

$: annotation_x1 =  (max_Column1 + rScale(prev2Intl_value))/2 + rScale(prev1Intl_value) + 2 * gap - rScale(annotation_value)/2;
$: annotation_y1 = (max_A - rScale(annotation_value))/2;

$: annotation_x2 = annotation_x1 + 15;
$: annotation_y2 = annotation_y1 - 10;

// $: annotation_cx = (annotation_x1 + annotation_x2) / 2;
// $: annotation_cy = annotation_y1 - 10;

$: console.log(processedData);

let showAnnotations = false;

$: if ($tweenedPoints && $tweenedPoints.length && !showAnnotations) {
  // Wait until tween animation duration (3000 ms) + delay
  setTimeout(() => {
    showAnnotations = true;
  }, 3100); // 3000 + a buffer
}
</script>

<main>
  <section>
  <div class="sticky">
    <div class="chart-container" bind:clientWidth={width}>
      <svg {width} {height}>
        <g class="inner-chart" transform="translate({margin.left}, {margin.top})">
          {#each processedData as d,i}
          <rect
          in:scale={{ duration: 600, delay: i * 40, start: 0.1, easing: cubicOut }}
          x={
          d.column=="Column_1"?
          (max_Column1 - rScale(d.intl_students))/2 : 
          d.column=="Column_2"?
          (max_Column1 + rScale(d.prev1Intl))/2 + gap:
          d.column=="Column_3"?
          (max_Column1 + rScale(d.prev2Intl))/2 + rScale(d.prev1Intl) + 2 * gap  
          :(max_Column1 + rScale(d.prev3Intl))/2 + + rScale(d.prev2Intl)+ rScale(d.prev1Intl) + 3 * gap
          }
          y={d.group=="A" ? 
          (max_A - rScale(d.intl_students))/2 : 
          d.group=="B" ? 
          max_A + row_width + (max_B - rScale(d.intl_students))/2 : 
          max_A + max_B + 2 * row_width + (max_C - rScale(d.intl_students))/2}
          width={rScale(d.intl_students)}
          height={rScale(d.intl_students)}
          stroke="none"
          stroke-width={1}
          fill={background_Color}
          opacity={0.5}
        />
        <text
        class="text-lables"
        x={
        d.column=="Column_1" && i != 0?
        max_Column1/2 - (10 * d.code.length)/2 + 2: 
        d.column=="Column_1" && i == 0?
        max_Column1/2 - (10 * d.code.length)/2 - 10:
        d.column=="Column_2"?
        (max_Column1 + rScale(d.prev1Intl))/2 + rScale(d.intl_students)/2 - (10 * d.code.length)/2 + gap + 2:
        d.column=="Column_3"?
        (max_Column1 + rScale(d.prev2Intl))/2 + rScale(d.prev1Intl) + 2 * gap + rScale(d.intl_students)/2 - (10 * d.code.length)/2 + 3 :
        (max_Column1 + rScale(d.prev3Intl))/2 + rScale(d.prev2Intl)+ rScale(d.prev1Intl) + 3 * gap + rScale(d.intl_students)/2 - (10 * d.code.length)/2 + 3
      }
        y={d.group=="A" && i != 0? 
        max_A/2 + 5: 
        d.group=="A" && i == 0?
        max_A/2 - rScale(d.intl_students) - 2:
        d.group=="B" ? 
        max_A + row_width + max_B/2 + 5 :
        d.group=="C" && i==10?
        max_A + max_B + 2 * row_width + max_C/2 - rScale(d.intl_students):
        max_A + max_B + 2 * row_width + max_C/2 + 5}
        style="font-size: {width < 350 ? '11px' : width > 590 ? '15px' : '13px'};"
        >
        {d.code}
        </text>
        {/each}
        <!-- {#if showAnnotations}
        <g transition:fade={{ duration: 1000 }}>
        <line
        x1={annotation_x1 + 0.75 * rScale(annotation_value)}
        y1={annotation_y1 + 5}
        x2={annotation_x1 + 0.75 * rScale(annotation_value)}
        y2={annotation_y2 + 5}
        stroke="black"
        stroke-width="1.5"
       />
        <text class="annotation" x={annotation_x1} y={annotation_y1 - 10} font-size="12px">
         international students
        </text>
        <line
        x1={annotation_x1 + 1.25 * rScale(annotation_value)}
        y1={annotation_y1 + rScale(annotation_value) + 5}
        x2={annotation_x1 + 1.25 * rScale(annotation_value)}
        y2={annotation_y2 + rScale(annotation_value) + 5}
        stroke="black"
        stroke-width="1.5"
        />
        <text class="annotation" x={annotation_x1 + 0.75 * rScale(annotation_value)} y={annotation_y1 + rScale(annotation_value) + 20} font-size="12px">
          from China
        </text>
        </g>
        {/if} -->
        {#each $tweenedPoints as d}
        <rect
        x={
        d.column=="Column_1"?
        (max_Column1 - rScale(d.intl_students))/2 : 
        d.column=="Column_2"?
        (max_Column1 + rScale(d.prev1Intl))/2 + gap:
        d.column=="Column_3"?
        (max_Column1 + rScale(d.prev2Intl))/2 + rScale(d.prev1Intl) + 2 * gap  
        :(max_Column1 + rScale(d.prev3Intl))/2 + + rScale(d.prev2Intl)+ rScale(d.prev1Intl) + 3 * gap
        }
        y={
        d.group=="A" ? 
        (max_A - rScale(d.intl_students))/2 + (1 - d.share_china)*rScale(d.intl_students) : 
        d.group=="B" ? 
        max_A + row_width + (max_B - rScale(d.intl_students))/2 + (1 - d.share_china) * rScale(d.intl_students)  : 
        max_A + max_B + 2 * row_width + (max_C - rScale(d.intl_students))/2 + (1 - d.share_china) * rScale(d.intl_students)
        }
        width={rScale(d.intl_students)}
        height={d.share_china * rScale(d.intl_students)}
        stroke="none"
        stroke-width={0}
        fill={share_Color}
        opacity={0.4}
      />
      {/each}
        </g>
      </svg>
    </div>
  </div>
  <Steps bind:currentStep/>
  </section>
  </main>
  
  <style>
  .sticky {
    position: sticky;
    z-index: 1;
    height:90vh;
    top:5vh;
    margin-bottom: 1rem;
    display: flex;
    align-items: center;
    justify-content: center;
    /* the three lines above is how a parent center its children */
  }
  .chart-container{
    height:100%;
    width:100%;
    max-width: 700px;
    max-height: 500px;
  }
  
  section {
    position: relative;
  }
  
  main{
    max-width: 700px;
    margin:0 auto;
  }
  
  .text-lables{
      font-family: Retina, sans-serif;
      text-transform: uppercase;
    }

    /* .annotation{
      font-family: Retina, sans-serif;
      font-size: 15px;
    } */

  </style>