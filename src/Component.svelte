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

  export let mappingTitle
  export let mappingDate
  export let mappingStart
  export let mappingEnd

  export let mappingTitle2
  export let mappingDate2
  export let mappingStart2
  export let mappingEnd2

  export let dataProvider
  export let dataProvider2

  export let mappingColor
  export let mappingColor2

  export let allday
  export let allday2

  export let headerOptionsStart
  export let headerOptionsCenter
  export let headerOptionsEnd

  const { styleable } = getContext("sdk")
  const component = getContext("component")

  let calendarEl
  let calendar

  const toEvents = (rows, title, date, start, end, color, allDay) =>
    (rows ?? []).map(row => ({
      title: row[title],
      start: row[start] ?? row[date],
      end: row[end],
      color,
      // Only force all-day when enabled, otherwise let FullCalendar infer it
      // from the value (date-only vs. date with time)
      ...(allDay ? { allDay: true } : {}),
      event: row,
    }))

  $: events = [
    ...toEvents(dataProvider?.rows, mappingTitle, mappingDate, mappingStart, mappingEnd, mappingColor ?? "#313131", allday),
    ...toEvents(dataProvider2?.rows, mappingTitle2, mappingDate2, mappingStart2, mappingEnd2, mappingColor2 ?? "#eb4034", allday2),
  ]

  // Events are served through a stable function source and refetched when the
  // data changes, so new rows don't require re-applying all calendar options
  let currentEvents = []
  const fetchEvents = (info, success) => success(currentEvents)
  $: {
    currentEvents = events
    calendar?.refetchEvents()
  }

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
  <div bind:this={calendarEl}></div>
</div>
