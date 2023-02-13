<script lang="ts">
  import { onMount } from "svelte";
  import type { PageData } from "./$types";
  import { fade } from "svelte/transition";
  import { writable, type Writable } from "svelte/store";
  import { Tab, TabGroup } from "@skeletonlabs/skeleton";

  import { currentUser, currentCourse } from "tutors-reader-lib/src/stores/stores";
  import LabTime from "$lib/time/LabTime.svelte";
  import InstructorLabTime from "$lib/time/InstructorLabTime.svelte";
  import InstructorCalendarTime from "$lib/time/InstructorCalendarTime.svelte";
  import NavBar from "$lib/navigators/NavBar.svelte";
  import CalendarTime from "$lib/time/CalendarTime.svelte";
  import { authService } from "tutors-reader-lib/src/services/auth-service";
  import { page } from "$app/stores";

  export let data: PageData;
  const storeTab: Writable<string> = writable("Labs");
  let pinBuffer = "";
  let instructorMode = false;
  let tabSet: number = 0;

  onMount(async () => {
    window.addEventListener("keydown", keypressInput);
    authService.checkAuth($currentCourse);
  });

  function keypressInput(e: KeyboardEvent) {
    pinBuffer = pinBuffer.concat(e.key);
    if (pinBuffer === data.ignorePin) {
      instructorMode = true;
    }
  }
</script>

<svelte:head>
  <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/ag-grid-community/styles/ag-grid.css" />
</svelte:head>

<NavBar title="{data.course.lo.title}" subTitle="{$currentUser?.name}" />

<div in:fade="{{ duration: 500 }}" class="bg-base-200 mt-3 ">
  <TabGroup selected="{storeTab}">
    <Tab bind:group="{tabSet}" name="Labs" value="{0}">Labs</Tab>
    <Tab bind:group="{tabSet}" name="LabsChart" value="{1}">LabsChart</Tab>
    <Tab bind:group="{tabSet}" name="Calendar" value="{2}">Calendar</Tab>

    {#if instructorMode}
      <Tab bind:group="{tabSet}" name="LabsAllStudent" value="{3}">Labs All Student</Tab>
      {#if data.course.hasEnrollment()}
        <Tab bind:group="{tabSet}" name="LabsAllStudent" value="{4}">Labs All Enrolled Student</Tab>
      {/if}
      <Tab bind:group="{tabSet}" name="allLabsChart" value="{5}">Labs All Students - Chart</Tab>
      <Tab bind:group="{tabSet}" name="allLabsChart" value="{6}">Calendar All Students</Tab>
    {/if}
  </TabGroup>
  {#if tabSet === 0}
    <LabTime user="{data.user}" allLabs="{data.allLabs}" chart="{false}" />
  {:else if tabSet === 1}
    <LabTime user="{data.user}" allLabs="{data.allLabs}" chart="{true}" />
  {:else if tabSet === 2}
    <CalendarTime user="{data.user}" calendarData="{data.calendar}" />
  {:else if tabSet === 3}
    <InstructorLabTime userMap="{data.users}" allLabs="{data.allLabs}" chart="{false}" />
  {:else if tabSet === 4}
    <InstructorLabTime userMap="{data.enrolledUsers}" allLabs="{data.allLabs}" chart="{false}" />
  {:else if tabSet === 5}
    <InstructorLabTime userMap="{data.users}" allLabs="{data.allLabs}" chart="{true}" />
  {:else if tabSet === 6}
    <InstructorCalendarTime userMap="{data.users}" calendarData="{data.calendar}" />
  {/if}
</div>
