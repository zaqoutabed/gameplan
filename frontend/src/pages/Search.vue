<template>
  <header class="sticky top-0 z-10 border-b bg-white px-4 py-2.5 sm:px-5">
    <div class="flex items-center justify-between">
      <PageBreadcrumbs
        :items="[{ label: 'Search', route: { name: 'Search' } }]"
      />
    </div>
  </header>
  <div class="mx-auto mt-6 max-w-4xl px-4 sm:px-5">
    <div class="flex items-center space-x-2">
      <TextInput
        class="w-full"
        placeholder="Type a keyword and hit enter to search"
        autocomplete="off"
        v-model="query"
        @keydown.enter="(e) => search(e.target.value)"
      >
        <template>
          <LucideSearch class="w-4 text-gray-600" />
        </template>
      </TextInput>
      <Button @click="search(query)" :loading="$resources.search.loading">
        Search
      </Button>
    </div>
    <div
      v-if="$resources.search.params && $resources.search.data"
      class="mt-4 text-base font-semibold text-gray-800"
    >
      About {{ $resources.search.data.total }} results for "{{
        $resources.search.params?.query
      }}" ({{ $resources.search.data.duration.toFixed(2) }}
      ms)
    </div>
    <div
      class="pb-10"
      v-if="$resources.search.params && $resources.search.data"
    >
      <div
        class="mt-5"
        v-for="group in $resources.search.data.groups"
        :key="group.title"
      >
        <div class="mb-3 text-base text-gray-600">
          {{ group.title }}
        </div>
        <router-link
          v-for="item in group.items"
          :key="item.name + item.via_comment"
          :to="getRoute(item)"
          class="block overflow-hidden rounded px-2.5 py-3 hover:bg-gray-100"
        >
          <div class="flex items-center">
            <div class="text-base font-medium" v-html="item.title" />
            <span class="px-1 leading-none text-gray-600"> &middot; </span>
            <div class="text-sm text-gray-600">
              {{ timestamp(item) }}
            </div>
          </div>
          <div
            v-if="item.content"
            class="mt-1 text-p-base text-gray-700"
            v-html="trimContent(item.content)"
          ></div>
        </router-link>
      </div>
    </div>
  </div>
</template>
<script>
import { TextInput } from 'frappe-ui'

export default {
  name: 'AppSearch',
  data() {
    return {
      query: '',
    }
  },
  mounted() {
    this.query = this.$route.query.q || ''
    this.search(this.query)
    this.$router.replace({ query: null })
  },
  resources: {
    search: {
      cache: 'Search',
      url: 'gameplan.search.search',
      makeParams(query) {
        return { query, start: this.start }
      },
      transform(data) {
        let out = {
          groups: [],
          total: data.total,
          duration: data.duration,
        }
        for (let doctype in data.results) {
          let group = null
          if (doctype === 'GP Discussion') {
            group = 'Discussions'
          } else if (doctype === 'GP Task') {
            group = 'Tasks'
          } else if (doctype === 'GP Page') {
            group = 'Pages'
          }
          if (!group) {
            continue
          }
          out.groups.push({
            title: group,
            items: data.results[doctype],
          })
        }
        return out
      },
    },
  },
  methods: {
    search(value) {
      if (value) {
        this.$resources.search.submit(value)
      }
    },
    trimContent(content) {
      let trimmedLength = 200
      let indexOf = content.indexOf('<mark>')
      if (indexOf === -1) {
        return content.slice(0, 200)
      }
      let start = indexOf - trimmedLength / 2
      if (start < 0) {
        start = 0
      }
      let end = indexOf + trimmedLength / 2
      if (end > content.length) {
        end = content.length
      }
      return content.slice(start, end)
    },
    timestamp(d) {
      let timestamp = d.modified
      if (this.$dayjs().diff(timestamp, 'day') < 25) {
        return this.$dayjs(timestamp).fromNow()
      }
      if (this.$dayjs().diff(timestamp, 'year') < 1) {
        return this.$dayjs(timestamp).format('D MMM')
      }
      return this.$dayjs(timestamp).format('D MMM YYYY')
    },
    getRoute(item) {
      if (item.doctype === 'GP Discussion') {
        return {
          name: 'ProjectDiscussion',
          params: {
            teamId: item.team,
            projectId: item.project,
            postId: item.name,
          },
        }
      }
      if (item.doctype === 'GP Task') {
        return {
          name: item.project ? 'ProjectTaskDetail' : 'HomeTask',
          params: {
            teamId: item.team,
            projectId: item.project,
            taskId: item.name,
          },
        }
      }
      if (item.doctype === 'GP Page') {
        return {
          name: 'ProjectPage',
          params: {
            teamId: item.team,
            projectId: item.project,
            pageId: item.name,
          },
        }
      }
      return { name: 'Search' }
    },
  },
  pageMeta() {
    return {
      title: 'Search',
      emoji: '🔍',
    }
  },
  components: { TextInput },
}
</script>
<style>
.search-result mark {
  border-radius: 0.25rem;
  background-color: theme('colors.yellow.200');
  padding: 2px 3px;
}
</style>
