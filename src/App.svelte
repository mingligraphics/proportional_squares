<script>
  import data from "./data/schools.json";
  import { scaleSqrt } from "d3-scale";
  import { extent, max, group } from 'd3-array';
  console.log(data);

  let width = 700;
  let height = 500;

  let background_Color = "#CED3DD";
  
  const margin = {
      top:0,
      right: 10,
      bottom: 0,
      left: 50
    };

  $: innerWidth = width - margin.left - margin.right;
  let innerHeight = height - margin.top - margin.bottom;

  $: students_range = extent(data, d => d.intl_students);

  $: console.log(students_range)

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

$: console.log(processedData)
</script>

<main>
  <section>
  <div class="sticky">
    <div class="chart-container" bind:clientWidth={width}>
      <svg {width} {height}>
        <g class="inner-chart" transform="translate({margin.left}, {margin.top})">
          {#each processedData as d,i}
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
          opacity={0.7}
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
        </g>
      </svg>
    </div>
  </div>
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
  </style>