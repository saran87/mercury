<template lang="pug">
  transition(name="fade" mode="out-in" enter-active-class="animated fadeIn" @after-enter="bootstrap" appear)
    .list(key="list" v-if="!this.inspecting")
      .ui.middle.aligned.selection.list
        .item(
            v-for="zk in zookeepers"
            :key="zk.hostname"
            @click="inspect(zk.hostname)"
        )
          .content
            .header
              | {{ zk.hostname }}
              template(v-if="zk.isLeader")
                span.fa-stack(data-content="This node is currently the leader" data-position="top center")
                  i.fa.fa-square-o.fa-stack-2x
                  i.fa.fa-star.fa-stack-1x
            .description Average Latency: #[b {{ zk.latency }}]
    .metrics(v-else key="individual")
      .centered-container(v-if="loading > 0")
        spinner.spinner(
          size="medium"
          line-fg-color="#E37D00"
          line-bg-color="#FFF"
        )
      template(v-else)
        h4.breadcrumb
          i.link.arrow.left.icon(@click="inspect(null)" title="Back to list")
          | {{ this.inspecting }}
        h3.no-metrics(v-if="!zookeeper.metrics")
          | No Metrics
        table.ui.sortable.very.compact.stackable.celled.table(v-else)
          thead
            tr
              th Metric
              th Value
          tbody
            tr(v-for="metric in zookeeper.metrics")
              td {{ metric.metric | mapMetric }}
              td {{ metric.value }}
</template>

<script>
  import Spinner from 'vue-simple-spinner';
  import gql from 'graphql-tag';

  export default {
    apollo: {
      zookeepers: {
        query: gql` query Zookeepers($address: String!) {
          cluster(address: $address) {
            zookeepers {
              hostname
              latency
              isLeader
            }
          }
        }`,
        variables() {
          return {
            address: this.$store.state.cluster
          };
        },
        update(data) {
          return data.cluster.zookeepers;
        },
        result(res) {
          if (!res.loading) {
            const zookeepers = res.data.cluster.zookeepers;
            this.$store.commit('updateZookeepers', zookeepers);
          }
        }
      },
      zookeeper: {
        query: gql` query Zookeeper($address: String!, $zookeeper: String!) {
          cluster(address: $address) {
            zookeeper(hostname: $zookeeper) {
              metrics
            }
          }
        }`,
        variables() {
          return {
            address: this.$store.state.cluster,
            zookeeper: this.inspecting
          };
        },
        update(data) {
          return data.cluster.zookeeper;
        },
        skip() {
          return !this.inspecting;
        }
      }
    },
    data() {
      return {
        inspecting: null,
        loading: 0
      };
    },
    methods: {
      inspect(zookeeper) {
        this.inspecting = zookeeper;
      },
      bootstrap() {
        $('.fa-stack').popup({ inline: false });
        $('#zookeepers .ui.table').tablesort();
      }
    },
    updated() {
      this.bootstrap();
    },
    filters: {
      mapMetric(metric) {
        const map = {
          zk_avg_latency: 'Average Latency',
          zk_max_latency: 'Max Latency',
          zk_min_latency: 'Min Latency',
          zk_packets_received: 'Packets Received',
          zk_packets_sent: 'Packets Sent',
          zk_num_alive_connections: 'Alive Connections',
          zk_outstanding_requests: 'Outstanding Requests',
          zk_server_state: 'Mode',
          zk_znode_count: 'Node Count',
          zk_watch_count: 'Watch Count',
          zk_ephemerals_count: 'Ephemerals Count',
          zk_approximate_data_size: 'Approximate Data Size',
          zk_open_file_descriptor_count: 'File Descriptors',
          zk_max_file_descriptor_count: 'Max File Descriptors',
          zk_followers: 'Follower Count',
          zk_synced_followers: 'Synchronized Followers',
          zk_pending_syncs: 'Pending Syncs',
          zk_version: 'Version'
        };

        return map[metric] || metric;
      }
    },
    components: { Spinner }
  };
</script>
