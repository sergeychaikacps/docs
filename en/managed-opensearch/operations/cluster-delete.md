---
title: "Deleting an {{ OS }} cluster"
description: "You can delete an {{ OS }} cluster if you no longer need it. All data in the cluster will be deleted. In the management console, select the folder you want to delete a cluster from."
keywords:
  - Creating an OpenSearch cluster
  - OpenSearch cluster
  - OpenSearch
---

# Deleting an {{ OS }} cluster

You can delete an {{ OS }} cluster if you no longer need it. All data in the cluster will be deleted.

## Before deleting a cluster {#before-you-delete}

* [Disable deletion protection](update.md) for the cluster if it is enabled.
* [Save the cluster ID](cluster-list.md#list-clusters).

{% include [backups-stored](../../_includes/mdb/backups-stored.md) %}

## Deleting a cluster {#delete}

{% list tabs group=instructions %}

- Management console {#console}

   1. In the [management console]({{ link-console-main }}), select the folder you want to delete a cluster from.
   1. Select **{{ ui-key.yacloud.iam.folder.dashboard.label_managed-opensearch }}**.
   1. Click ![image](../../_assets/console-icons/ellipsis.svg) for the cluster you need and select **{{ ui-key.yacloud.mdb.cluster.overview.button_action-delete }}**.
   1. Confirm cluster deletion and click **{{ ui-key.yacloud.mdb.dialogs.popup-confirm-delete-cluster_button }}**.

- API {#api}

   To delete a cluster, use the [delete](../api-ref/Cluster/delete.md) REST API method for the [Cluster](../api-ref/Cluster/index.md) resource or the [ClusterService/Delete](../api-ref/grpc/cluster_service.md#Delete) gRPC API call and provide the ID of the cluster to be deleted in the `clusterId` request parameter.

   {% include [get-cluster-id](../../_includes/managed-opensearch/get-cluster-id.md) %}

{% endlist %}
