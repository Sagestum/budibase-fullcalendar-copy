<script>
  import { getContext, onMount } from "svelte"
  import { Calendar } from "fullcalendar"
  import dayGridPlugin from "fullcalendar/daygrid"
  import timeGridPlugin from "fullcalendar/timegrid"
  import listPlugin from "fullcalendar/list"
  import classicThemePlugin from "fullcalendar/themes/classic"
  import allLocales from "fullcalendar/locales-all"
  import "fullcalendar/skeleton.css"
  import "fullcalendar/themes/classic/theme.css"
  import "fullcalendar/themes/classic/palette.css"

  export let language
  export let calendarEvent

  export let headerOptionsStart
  export let headerOptionsCenter
  export let headerOptionsEnd

  export let weekendHighlight
  export let weekendColor

  // Event group settings (dataProvider, mappingTitle, ...) are read from
  // $$props: group 1 has no suffix, group n uses the suffix n
  const MAX_GROUPS = 6
  const DEFAULT_COLORS = ["#313131", "#eb4034", "#2e7d32", "#ef6c00", "#6a1b9a", "#00838f"]

  const { styleable } = getContext("sdk")
  const component = getContext("component")

  let calendarEl
  let calendar

  const groupEvents = (props, group) => {
    const suffix = group === 1 ? "" : group
    const setting = key => props[`${key}${suffix}`]
    const rows = setting("dataProvider")?.rows ?? []
    const color = setting("mappingColor") ?? DEFAULT_COLORS[group - 1]
    return rows.map(row => ({
      title: row[setting("mappingTitle")],
      start: row[setting("mappingStart")] ?? row[setting("mappingDate")],
      end: row[setting("mappingEnd")],
      color,
      // Only force all-day when enabled, otherwise let FullCalendar infer it
      // from the value (date-only vs. date with time)
      ...(setting("allday") ? { allDay: true } : {}),
      event: row,
    }))
  }

  // Components saved before the group count setting existed have no value,
  // so they keep rendering every configured group
  $: groupCount = Math.min(parseInt($$props.groupCount) || MAX_GROUPS, MAX_GROUPS)
  $: events = Array.from({ length: groupCount }, (_, i) =>
    groupEvents($$props, i + 1)
  ).flat()

  // Events are served through a stable function source and refetched when the
  // data changes, so new rows don't require re-applying all calendar options
  let currentEvents = []
  const fetchEvents = (info, success) => success(currentEvents)
  $: {
    currentEvents = events
    calendar?.refetchEvents()
  }

  // Marks Saturday and Sunday; today keeps the theme's own highlight
  const weekendClass = info =>
    (info.dow === 0 || info.dow === 6) && !info.isToday ? "fc7-weekend" : ""

  $: options = {
    plugins: [dayGridPlugin, listPlugin, timeGridPlugin, classicThemePlugin],
    headerToolbar: {
      start: headerOptionsStart,
      center: headerOptionsCenter,
      end: headerOptionsEnd,
    },
    locales: allLocales,
    locale: language,
    dayMaxEvents: true,
    eventClick: info => {
      calendarEvent?.({
        value: info.event,
      })
    },
    events: fetchEvents,
    eventColor: "#378006",
    ...(weekendHighlight
      ? {
          dayHeaderClass: weekendClass,
          dayCellClass: weekendClass,
          dayLaneClass: weekendClass,
        }
      : {}),
  }

  // Keep the calendar in sync when settings change
  $: calendar?.resetOptions(options)

  onMount(() => {
    calendar = new Calendar(calendarEl, options)
    calendar.render()
    return () => calendar.destroy()
  })
</script>

<div use:styleable={$component.styles}>
  <div bind:this={calendarEl} style:--fc7-weekend={weekendColor || "#f2f2f2"}></div>
</div>

<style>
  div :global(.fc7-weekend) {
    background-color: var(--fc7-weekend);
  }
</style>
