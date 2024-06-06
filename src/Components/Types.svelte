<script>
  import { onMount } from 'svelte';
  import * as d3 from 'd3';

  let selectedType = 'Type 1';

  const diabetesTypes = ['Type 1', 'Type 2', 'Gestational'];

  const typeData = {
    'Type 1': {
      characteristics: 'Autoimmune destruction of insulin-producing beta cells',
      symptoms: ['Frequent urination', 'Increased thirst', 'Weight loss'],
      causes: 'Genetic and environmental factors',
      treatments: ['Insulin injections', 'Diet and exercise'],
      imageURL: 'https://www.premierintegratedlabs.com.my/wp-content/uploads/2020/11/121120-Diabetes-Type-1-Social-Media-1024x1024.jpg'
    },
    'Type 2': {
      characteristics: 'Insulin resistance and relative insulin deficiency',
      symptoms: ['Frequent urination', 'Increased thirst', 'Fatigue'],
      causes: 'Genetic and lifestyle factors',
      treatments: ['Oral medications', 'Diet and exercise', 'Insulin injections'],
      imageURL: 'https://www.premierintegratedlabs.com.my/wp-content/uploads/2020/11/121120-Diabetes-Type-2-Social-Media.jpg'
    },
    'Gestational': {
      characteristics: 'Diabetes diagnosed during pregnancy',
      symptoms: ['Frequent urination', 'Increased thirst', 'Fatigue'],
      causes: 'Hormonal changes during pregnancy',
      treatments: ['Diet and exercise', 'Insulin injections if necessary'],
      imageURL: 'https://www.premierintegratedlabs.com.my/wp-content/uploads/2020/11/121120-Gestational-Diabetes-Social-Media-1024x1024.jpg'
    }
  };

  onMount(() => {
    updateChart();
  });

  $: if (selectedType) {
    updateChart();
  }

  function updateChart() {
    d3.select('#chart').selectAll('*').remove();

    const width = 800;
    const height = 400;
    const margin = { top: 20, right: 30, bottom: 40, left: 100 };

    const svg = d3.select('#chart').append('svg')
      .attr('width', width + margin.left + margin.right)
      .attr('height', height + margin.top + margin.bottom)
      .append('g')
      .attr('transform', `translate(${margin.left},${margin.top})`);

    const data = typeData[selectedType];

    svg.append('image')
      .attr('xlink:href', data.imageURL)
      .attr('width', width)
      .attr('height', height);
  }
</script>

<h1 class="body-header">Types of Diabetes</h1>
<p class="body-text">
  There are three main types of diabetes: Type 1, Type 2, and Gestational Diabetes.
  <br>
  <br>
  - Type 1 Diabetes is an autoimmune condition where the body's immune system attacks and destroys insulin-producing 
  beta cells in the pancreas. This type typically develops in children and young adults
  and requires lifelong insulin therapy.
  <br>
  <br>
 - Type 2 Diabetes is characterized by insulin resistance and a gradual decline in insulin production. 
  It is the most common form of diabetes, often associated with obesity and a sedentary lifestyle. 
  Management includes lifestyle changes, oral medications, and sometimes insulin therapy.
  <br>
  <br>
  - Gestational Diabetes occurs during pregnancy when hormonal changes cause insulin resistance. 
  Although it usually resolves after childbirth, it increases the risk of developing 
  Type 2 Diabetes later in life for both the mother and the child.
  <br>
  <br>
  Understanding these distinctions is crucial for effective management and treatment. 
  The following interactive comparison chart allows you to explore these differences in greater detail.
  <br>
  <br>
</p>

<style>
  .symptom-image {
      cursor: pointer;
  }

  svg {
      border: 1px solid #ccc;
  }

  #chart {
      display: flex;
      justify-content: center;
      margin-top: 20px;
  }

  .centered {
      display: flex;
      justify-content: center;
      align-items: center;
      flex-direction: column;
  }

  table {
      margin: 20px auto;
  }
</style>

<div class="centered">
  <h2>Select Diabetes Type:</h2>
  <select id="type" bind:value={selectedType}>
      {#each diabetesTypes as type}
          <option value={type}>{type}</option>
      {/each}
  </select>
</div>

<div id="chart" class="centered"></div>

<table>
  <thead>
      <tr>
          <th>Characteristic</th>
          <th>Details</th>
      </tr>
  </thead>
  <tbody>
      <tr>
          <td>Definition: </td>
          <td>{typeData[selectedType].characteristics}</td>
      </tr>
      <tr>
          <td>Symptoms: </td>
          <td>{typeData[selectedType].symptoms.join(', ')}</td>
      </tr>
      <tr>
          <td>Causes: </td>
          <td>{typeData[selectedType].causes}</td>
      </tr>
      <tr>
          <td>Treatments: </td>
          <td>{typeData[selectedType].treatments.join(', ')}</td>
      </tr>
  </tbody>
</table>
