<script>
  import { createEventDispatcher } from 'svelte';

  const dispatch = createEventDispatcher();

  export let month;
  export let year;
  export let start;
  export let end;
  export let canIncrementMonth;
  export let canDecrementMonth;
  export let canIncrementYear;
  export let canDecrementYear;
  export let monthsOfYear;

  let monthSelectorOpen = false;
  let yearSelectorOpen = false;
  let availableMonths;
  let availableYears;
  let startYear;
  let endYear;

  $: {
	startYear = start.getFullYear();
	endYear = end.getFullYear();

    let isOnLowerBoundary = startYear === year;
    let isOnUpperBoundary = endYear === year;
    availableMonths = monthsOfYear.map((m, i) => {
      return Object.assign({}, {
        name: m[0],
        abbrev: m[1]
      }, {
        selectable:
          (!isOnLowerBoundary && !isOnUpperBoundary)
          || (
            (!isOnLowerBoundary || i >= start.getMonth())
            && (!isOnUpperBoundary || i <= end.getMonth())
          )
      });
	});

	let yearRangeLength = endYear - startYear;
	let yearRange = [];
	for (let i = 0; (startYear + i <= endYear); i++) {
		yearRange[i] = startYear + i;
	}
	// We want to show years 12 at a time, but we only want to show the years which are included in the range (between startYear and endYear)
	availableYears = yearRange.filter(function(y) {
	  // TESTING: SKIPPING
	  // return true;
	  let distanceFromUpperBoundary = endYear - y;
	  let distanceFromLowerBoundary = y - startYear;
	  let lowerRange = distanceFromLowerBoundary < 6 ? distanceFromLowerBoundary : 6; // 11 instead of 12 because we are counting the currently selected year as well
	  let upperRange = distanceFromUpperBoundary < 6 ? distanceFromUpperBoundary : 6; // Ditto
	 // TESTING 
	  console.log(y===year?"CURRENT YEAR":"            ","y",y,"year",year,"lowerRange",lowerRange,"upperRange",upperRange,"distanceFromLowerBoundary",distanceFromLowerBoundary,"distanceFromUpperBoundary",distanceFromUpperBoundary);
	  if ( // FIXME: ? 
		  y === year || // If current year
		  (y >= year - lowerRange && y <= year + upperRange) // If within the 12 year range
	   ) {
		   return true;
	   } else {
		   return false;
	   }
	}).map((y, i) => {
	  return Object.assign({}, {
        name: y,
        abbrev: y
      }, {
        selectable:
          (!isOnLowerBoundary && !isOnUpperBoundary)
          || (
            (!isOnLowerBoundary || y >= start.getFullYear())
            && (!isOnUpperBoundary || y <= end.getFullYear())
          )
	  });
	});
	console.log(availableYears);

  }

  function toggleMonthSelectorOpen() {
	yearSelectorOpen = false;
    monthSelectorOpen = !monthSelectorOpen;
  }

  function toggleYearSelectorOpen() {
	monthSelectorOpen = false;
    yearSelectorOpen = !yearSelectorOpen;
  }

  function monthSelected(event, { m, i }) {
    event.stopPropagation();
    if (!m.selectable) return;
    dispatch('monthSelected', i);
    toggleMonthSelectorOpen();
  }

  function yearSelected(event, { y, i }) {
    event.stopPropagation();
    if (!y.selectable) return;
	dispatch('yearSelected', y.name);
    toggleYearSelectorOpen();
  }
</script>

<div class="title">
  <div class="heading-section">
    <div class="control" 
      class:enabled={!yearSelectorOpen ? canDecrementMonth : canDecrementYear}
      on:click={() => !yearSelectorOpen ? dispatch('incrementMonth', -1) : dispatch('incrementYear', -1) }>
      <i class="arrow left"></i>
    </div>
    <div class="label">
      <span on:click={toggleMonthSelectorOpen}>{monthsOfYear[month][0]}</span> <span on:click={toggleYearSelectorOpen}>{year}</span>
    </div> 
    <div class="control"
      class:enabled={!yearSelectorOpen ? canIncrementMonth : canIncrementYear}
      on:click={() => !yearSelectorOpen ? dispatch('incrementMonth', 1) : dispatch('incrementYear', 1) }>
      <i class="arrow right"></i>
    </div>
  </div>
  <div class="month-selector" class:open={monthSelectorOpen}>
    {#each availableMonths as monthDefinition, index}
      <div 
        class="month-selector--month" 
        class:selected={index === month}
        class:selectable={monthDefinition.selectable}
        on:click={e => monthSelected(e, { m: monthDefinition, i: index })}
      >
        <span>{monthDefinition.abbrev}</span>
      </div>
    {/each}
  </div>
  <div class="year-selector" class:open={yearSelectorOpen}>
    {#each availableYears as yearDefinition, index}
      <div 
        class="year-selector--year" 
        class:selected={startYear + index === year}
        class:selectable={yearDefinition.selectable}
        on:click={e => yearSelected(e, { y: yearDefinition, i: startYear + index })}
      >
        <span>{yearDefinition.abbrev}</span>
      </div>
    {/each}
  </div>
</div>

<style>
  .heading-section { 
    font-size: 20px;
    padding: 24px 15px;
    display: flex;
    justify-content: space-between;
    color: #3d4548;
    font-weight: bold;
  }
  .label { 
    cursor: pointer;
  }
  .month-selector, .year-selector { 
    position: absolute;
    top: 75px; 
    left: 0; 
    right: 0; 
    bottom: 0; 
    background-color: #fff;
    transition: all 300ms; 
    transform: scale(1.2); 
    opacity: 0; 
    visibility: hidden;
    z-index: 1;
    text-align: center;
  }
  .month-selector.open, .year-selector.open { 
    transform: scale(1); 
    visibility: visible;
    opacity: 1;
  }
  .month-selector--month, .year-selector--year { 
    width: 31.333%; 
    margin: .5%; 
    height: 23%;
    display: inline-block;
    color: #4a4a4a;
    border: 1px solid #efefef;
    opacity: 0.2;
  }
  .month-selector--month.selectable, .year-selector--year.selectable { 
    opacity: 1; 
  }
  .month-selector--month.selectable:hover, .year-selector--year.selectable:hover { 
    cursor: pointer;
    box-shadow: 0px 0px 3px rgba(0,0,0,0.15);
  }
  .month-selector--month.selected, .year-selector--year.selected { 
    background: var(--highlight-color);
    color: #fff;
  }
  .month-selector--month:before, .year-selector--year:before { 
    content: ' ';
    display: inline-block;
    height: 100%;
    vertical-align: middle;
  }
  .month-selector--month span, .year-selector--year span { 
    vertical-align: middle; 
    display: inline-block;
  }

  .control { 
    padding: 0 8px;
    opacity: 0.2;
    transform: translateY(3px);
  }

  .control.enabled { 
    opacity: 1; 
    cursor: pointer;
  }

  .arrow {
    display: inline-block;
    width: 18px;
    height: 18px;
    border-style: solid;
    border-color: #a9a9a9;
    border-width: 0;
    border-bottom-width: 2px;
    border-right-width: 2px;
  }

  .arrow.right {
    transform: rotate(-45deg);
    -webkit-transform: rotate(-45deg);
  }

  .arrow.left {
    transform: rotate(135deg);
    -webkit-transform: rotate(135deg);
  }

</style>
