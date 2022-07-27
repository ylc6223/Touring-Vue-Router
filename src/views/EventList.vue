<template>
  <h1>Events for Good</h1>
  <div class="events">
    <EventCard v-for="event in events" :key="event.id" :event="event" />

    <div class="pagination">
      <router-link
        id="page-prev"
        :to="{ name: 'EventList', query: { page: page - 1 } }"
        rel="prev"
        v-if="page != 1"
        >&#60; Previous
      </router-link>

      <router-link
        id="page-next"
        :to="{ name: 'EventList', query: { page: page + 1 } }"
        rel="next"
        v-if="hasNextPage"
        >Next &#62;
      </router-link>
    </div>
  </div>
</template>

<script>
import EventCard from '@/components/EventCard.vue'
import EventService from '@/services/EventService.js'
import NProgress from 'nprogress'
export default {
  name: 'EventList',
  props: ['page'],
  components: {
    EventCard
  },
  data() {
    return {
      events: null,
      totalEvents: 0,
      unsavedChanges: true
    }
  },
  computed: {
    hasNextPage() {
      var totalPages = Math.ceil(this.totalEvents / 2)

      return this.page < totalPages
    }
  },
  /* eslint-disable */
  //进入之前 这里无法拿到this
  // beforeRouteEnter 是支持给 next 传递回调的唯一守卫
  beforeRouteEnter(to, from, next) {
    NProgress.start()
    return EventService.getEvents(2, to.query.page || 1)
      .then(response => {
        next(vm => {
          vm.events = response.data
          vm.totalEvents = response.headers['x-total-count']
        })
      })
      .catch(() => {
        //这里因为不能访问this 所以不能够使用this.$router.push方法了
        next({ name: 'NetworkError' })
      })
      .finally(() => {
        NProgress.done()
      })
  },
  //路由更新之前
  beforeRouteUpdate(to, from) {
    NProgress.start()
    return EventService.getEvents(2, this.page)
      .then(response => {
        this.events = response.data
        this.totalEvents = response.headers['x-total-count']
        // next()
      })
      .catch(() => {
        //这里因为不能访问this 所以不能够使用this.$router.push方法了
        // next({ name: 'NetworkError' })
        return {name:"NetworkError"}
      })
      .finally(() => {
        NProgress.done()
      })
  },
  // vue-router4.x写法 next参数可选可不选,如果写了就一定要进行调用
  //离开之前
  beforeRouteLeave(routeTo, routeFrom) {
    if (this.unsavedChanges) {
      const answer = window.confirm(
        'Do you really want to leave? You have unsaved changes!'
      )
      if (!answer) {
        return false // <-- Cancels the navigation
      }
    }
  }

  // beforeRouteLeave(routeTo, routeFrom, next) {
  //   if (this.unsavedChanges) {
  //     const answer = window.confirm(
  //         'Do you really want to leave? You have unsaved changes!'
  //     )
  //     if (answer) {
  //       next() // <-- Confirms the navigation
  //     } else {
  //       next(false) // <-- Cancels the navigation
  //     }
  //   } else {
  //     next() // <-- Confirms the navigation
  //   }
  // }
  /* eslint-disable */
}
</script>

<style scoped>
.events {
  display: flex;
  flex-direction: column;
  align-items: center;
}

.pagination {
  display: flex;
  width: 290px;
}

.pagination a {
  flex: 1;
  text-decoration: none;
  color: #2c3e50;
}

#page-prev {
  text-align: left;
}

#page-next {
  text-align: right;
}
</style>
