<template lang="pug">
  .topic-list
    .title
      i.fa.fa-send.symbol
      | Topics
    template(v-if="cluster")
      spinner.spinner(
        v-if="loading > 0"
        size="medium"
        line-fg-color="#E37D00"
        line-bg-color="#262B33"
      )
      template(v-else)
        .ui.fluid.search
          .ui.icon.input
            input(type="text" placeholder="Search Topics..." v-model="query")
            i.search.icon
        .ui.middle.aligned.inverted.selection.list
          .item(
              v-for="topic in filteredTopics"
              v-on:click="setTopic(topic)"
              :key="topic.name"
              :class="{ active: active(topic.name) }"
          )
            .content
              .header
                | {{ topic.name }}
                i.check.icon(v-if="active(topic.name)")
              .description Partitions: #[b {{ topic.partitions }}]
    .no.cluster.content(v-else)
      .hint
        | Select a Cluster
        i.fa.fa-arrow-up
</template>

<script>
  import Spinner from 'vue-simple-spinner';
  import search from 'fuzzysearch';
  import gql from 'graphql-tag';

  export default {
    apollo: {
      topics: {
        query: gql`query Topics($address: String!) {
          cluster(address: $address) {
            topics {
              name
              partitions
            }
          }
        }`,
        variables() {
          return {
            address: this.$store.state.cluster
          };
        },
        skip() {
          return !this.cluster;
        },
        update(data) {
          return data.cluster.topics;
        },
        result(res) {
          if (!res.loading) {
            const topics = res.data.cluster.topics;
            this.$store.commit('updateTopics', topics);
          }
        }
      }
    },
    data() {
      return {
        query: '',
        loading: 0
      };
    },
    computed: {
      filteredTopics() {
        if (!this.topics) return [];
        return this.topics.filter(({ name }) => search(this.query, name));
      },
      topic() {
        return this.$store.state.topic;
      },
      cluster() {
        return this.$store.state.cluster;
      }
    },
    methods: {
      setTopic(topic) {
        if (this.topic && (this.topic.name === topic.name)) {
          topic = null;
        }

        this.$store.commit('changeTopic', topic);
      },
      active(topic) {
        if (!this.topic) return false;
        return this.topic.name === topic;
      }
    },
    components: { Spinner }
  };
</script>

<style lang="scss" scoped>
  .topic-list {
    .spinner { padding: 0 0 1em }
    .ui.input {
      width: 100%;
      padding: 0 0.5em;
      font-size: 0.8em;
    }
    .ui.search .prompt {
      border-radius: 0;
    }
    .check.icon {
      float: right;
      color: #85CC18;
    }
    .content .description {
      font-size: 0.9em;
    }
    .list-enter-active, .list-leave-active {
      transition: all 0.5s !important;
    }
    .list-enter, .list-leave-to {
      opacity: 0;
      transform: translateY(30px);
    }
    .ui.list {
      overflow-y: auto;
      overflow-x: hidden; 
      .item { margin-right: 2px }
      .header { word-break: break-all }
    }
  }
</style>
