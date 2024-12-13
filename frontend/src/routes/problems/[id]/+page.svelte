<script lang="ts">
    import type { PageData } from './$types';
    import { browser } from '$app/environment';
    import MonacoEditor from '$lib/components/MonacoEditor.svelte';
    import { onMount, onDestroy } from 'svelte';
    import * as Resizable from "$lib/components/ui/resizable";
    import ResizableHandle from '$lib/components/ui/resizable/resizable-handle.svelte';
    
    export let data: PageData;
    let { problem } = data;
    
    let editorComponent: MonacoEditor;
    
    const languages = [
      { value: 'python', label: 'Python' }
    ];

    let selectedLanguage = languages[0].value; 
    let code = problem.code[selectedLanguage];
    let executionResult: { output: string; passed: number; total: number } | null = null;
    let error: string | null = null;

    // function handleLanguageChange(event: Event) {
    //   const target = event.target as HTMLSelectElement;
    //   selectedLanguage = target.value;
    //   code = problem.starterCode[selectedLanguage];
    // }
    
    async function handleSubmit() {
      const submittedCode = editorComponent.getCode();
      console.log('Submitted code:\n', submittedCode);
      console.log('Language:', selectedLanguage);
      
      try {
        const response = await fetch('http://localhost:3000/execute', {
          method: 'POST',
          headers: {
            'Content-Type': 'application/json'
          },
          body: JSON.stringify({
            code: submittedCode,
            language: selectedLanguage,
            testCases: problem.testCases,
            problemID: problem.id
          })
        });

        if (!response.ok) {
          throw new Error('Execution failed');
        }

        const result = await response.json();
        console.log('Execution result:', result);
        executionResult = result;
        error = null;
        problem.passed = result.passed;
      } catch (err) {
        console.error('Error executing code:', err);
        error = err instanceof Error ? err.message : String(err);
        executionResult = null;
      }
    }

    async function saveCode() {
      const submittedCode = editorComponent.getCode();
      console.log('Submitted code:\n', submittedCode);
      console.log('Language:', selectedLanguage);
      
      try {
        const response = await fetch('http://localhost:3000/save', {
          method: 'POST',
          headers: {
            'Content-Type': 'application/json'
          },
          body: JSON.stringify({
            code: submittedCode,
            problemID: problem.id,
            language: selectedLanguage,
          })
        });
      } catch (err) {
        console.error('Error saving code:', err);
        error = err instanceof Error ? err.message : String(err);
      }
    }
</script>

<!-- Add later when multi-language support is working -->
<!-- <div>
  <label for="language-select">Select Language:</label>
  <select id="language-select" on:change={handleLanguageChange}>
    {#each languages as lang}
      <option value={lang.value}>{lang.label}</option>
    {/each}
  </select>
</div> -->
<div class="h-[85vh]">
<Resizable.PaneGroup direction="horizontal" class="rounded-md border-2">
  <Resizable.Pane defaultSize={40}>
    <h1 class="text-3xl">{problem.title}</h1>
    <h2 class="pb-8">Difficulty: {problem.difficulty}</h2>
    <p class="pb-8">{problem.description}</p>
    <h2 class="text-2xl">Test Cases</h2>
    <ul>
      {#each problem.testCases as testCase}
        <li>
          <p>Input: {testCase.input}</p>
          <p>Output: {testCase.output}</p>
        </li>
      {/each}
    </ul>
  </Resizable.Pane>
  <ResizableHandle />
  <Resizable.Pane defaultSize={60}>
    <Resizable.PaneGroup direction="vertical">
      <Resizable.Pane defaultSize={100}>
        <MonacoEditor
          bind:this={editorComponent}
          bind:code={code}
          language={selectedLanguage}
        />
      </Resizable.Pane>
      <Resizable.Handle />
      <Resizable.Pane defaultSize={100}>
        <button on:click={handleSubmit}>Submit Solution</button>
        <button on:click={saveCode}>Save Code</button>
        {#if executionResult}
          <div>
            <h2>Execution Result</h2>
            <p>Tests Passed: {executionResult.passed} / {executionResult.total}</p>
            <pre>{executionResult.output}</pre>
          </div>
        {/if}
        {#if error}
          <div class="error">
            <p>Error: {error}</p>
          </div>
        {/if}
      </Resizable.Pane>
    </Resizable.PaneGroup>
  </Resizable.Pane>  
</Resizable.PaneGroup>
</div>



