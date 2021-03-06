<template lang="pug">
  .vuecal__cell(:class="[cssClass, splits.length ? 'splitted' : '']")
    .vuecal__cell-content(:class="splits.length && `vuecal__cell-split ${splits[i - 1].class}`" v-for="i in (splits.length || 1)")
      .split-label(v-if="splits.length" v-html="splits[i - 1].label")
      div(v-if="content" v-html="content")
      div(v-else)
        .vuecal__no-event(v-if="!cellEvents.length") {{ texts.noEvent }}
        .vuecal__event(:class="event.class" v-else v-for="(event, j) in (splits.length ? splitEvents[i] : events)" :key="j" :style="eventPosition(event, i)")
          .vuecal__event-title {{ event.title }}
          .vuecal__event-time
            | {{ event.startTime }}
            span(v-if="event.endTime") &nbsp;- {{ event.endTime }}
          .vuecal__event-content(v-html="event.content")
</template>

<script>
export default {
  props: {
    cssClass: {
      type: String,
      default: ''
    },
    date: {
      type: Date,
      required: true
    },
    events: {
      type: Array,
      default: () => []
    },
    content: {
      type: [String, Number],
      default: ''
    },
    splits: {
      type: Array,
      default: () => []
    }
  },
  data: () => ({
    splitEvents: [],
    // For each event, compare with others if overlapping.
    // Keep track of what is already check in this indexed array not to redo the check.
    comparedEvents: {}
  }),

  methods: {
    eventPosition (event, split = 0) {
      const timeCellHeight = parseInt(this.$parent.timeCellHeight)
      const timeStep = parseInt(this.$parent.timeStep)

      let minutesFromTop = event.startTimeMinutes - this.$parent.timeFrom
      const top = Math.round(minutesFromTop * timeCellHeight / timeStep)

      minutesFromTop = event.endTimeMinutes - this.$parent.timeFrom
      const bottom = Math.round(minutesFromTop * timeCellHeight / timeStep)

      const height = bottom - top

      return {
        top: top + 'px',
        height: height + 'px',
        minHeight: height + 'px'
      }
    },
    checkOverlappingEvents (event) {
      (this.splits.length && event.split ? this.splitEvents[event.split] : this.events).forEach(evt => {
        // Don't compare with itself or with already compared item.
        if (event.id !== evt.id && this.comparedEvents[event.id].indexOf(evt.id) === -1 && (this.comparedEvents[evt.id] || []).indexOf(event.id) === -1) {
          const eventStartsFirst = event.startTimeMinutes > evt.startTimeMinutes

          if ((eventStartsFirst && event.endTimeMinutes > evt.startTimeMinutes) ||
              (event.startTimeMinutes < evt.startTimeMinutes && event.endTimeMinutes > evt.startTimeMinutes)) {
            evt.class += eventStartsFirst ? ' overlapped' : ' overlapping'
            event.class += eventStartsFirst ? ' overlapping' : ' overlapped'
          }
        }

        this.comparedEvents[event.id].push(evt.id)
      })
    }
  },

  computed: {
    texts () {
      return this.$parent.texts
    },
    cellEvents () {
      this.comparedEvents = {}

      return this.events.map(event => {
        this.comparedEvents[event.id] = []

        // Only for splits.
        if (this.splits.length && event.split) {
          if (!this.splitEvents[event.split]) this.splitEvents[event.split] = []
          this.splitEvents[event.split].push(event)
        }

        this.checkOverlappingEvents(event)

        return event
      })
    }
  }
}
</script>

<style lang="scss">
.vuecal__cell {
  width: 100%;
  display: flex;
  justify-content: center;
  align-items: center;
  text-align: center;
  position: relative;
  .vuecal--month-view &,
  .vuecal--week-view &,
  .vuecal--day-view & {
    width: 14.2857%;
  }

  .vuecal--hide-weekends.vuecal--month-view &,
  .vuecal--hide-weekends.vuecal--week-view &,
  .vuecal--hide-weekends.vuecal--day-view & {
    width: 20%;
  }

  .vuecal--years-view & {width: 20%;}
  .vuecal--year-view & {width: 33.33%;}
  // .vuecal--month-view & {}
  // .vuecal--week-view & {}
  .vuecal--day-view & {flex: 1;}

  .click-to-navigate & {cursor: pointer;}
  .view-with-time & {display: block;}

  &.splitted {
    flex-direction: row;
    display: flex;
  }

  .vuecal__cell-split {
    display: flex;
    flex-grow: 1;
    flex-direction: column;
    height: 100%;
    position: relative;
  }

  &:before {
    position: absolute;
    top: 0;
    left: 0;
    right: -1px;
    bottom: -1px;
    border: 1px solid #ddd;
    content: '';
  }

  &.today,
  &.current {
    background-color: rgba(240, 240, 255, 0.4);
    z-index: 1;
  }

  &.selected {
    background-color: rgba(235, 255, 245, 0.4);
    z-index: 2;

    &:before {
      border-color: rgba(66, 185, 131, 0.5);
    }
  }

  &.out-of-scope {
    color: #ccc;
  }
}

.vuecal__no-event {
  padding-top: 1em;
  color: #aaa;
  user-select: none;
}

.vuecal__event {
  padding: 0.3em;
  color: #666;
  background-color: #f8f8f8;

  .view-with-time & {
    position: absolute;
    left: 0;
    right: 0;
    overflow: hidden;

    &:hover {
      height: auto !important;
    }
  }

  &.overlapped {right: 20%;}
  &.overlapping {left: 30%;box-shadow: 0 0 5px rgba(#000, 0.2);}
}
</style>
