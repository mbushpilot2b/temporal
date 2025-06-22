# State-Based Cache Eviction

## Overview

The state-based cache eviction feature allows Temporal to automatically evict workflows from the memory cache when they transition out of a running state (completed, terminated, failed, etc.). This helps reduce memory usage by ensuring only active workflows remain cached.

## Configuration

The feature is controlled by the dynamic configuration parameter:

```yaml
history.cacheStateBasedEviction: true
```

**Default value:** `false` (disabled by default)

## How It Works

1. **Normal Operation**: When `history.cacheStateBasedEviction` is `false`, workflows are evicted from cache based on TTL and LRU policies only.

2. **State-Based Eviction**: When `history.cacheStateBasedEviction` is `true`, the cache will additionally check the workflow's execution state during the release operation.

3. **Eviction Criteria**: Workflows are evicted immediately if they are in any of the following states:
   - `WORKFLOW_EXECUTION_STATE_COMPLETED`
   - `WORKFLOW_EXECUTION_STATE_ZOMBIE`
   - `WORKFLOW_EXECUTION_STATE_CORRUPTED`

4. **Running Workflows**: Workflows in `WORKFLOW_EXECUTION_STATE_RUNNING` remain in cache and follow normal TTL/LRU eviction policies.

## Benefits

- **Memory Efficiency**: Reduces memory footprint by removing completed workflows from cache immediately
- **Resource Optimization**: Allows cache space to be used more effectively for active workflows
- **Configurable**: Can be enabled/disabled via dynamic configuration without service restart

## Monitoring

The feature includes observability through:

### Metrics
- `cache_state_based_evictions`: Counter tracking the number of workflows evicted due to state transitions

### Logs
- Info-level logs when workflows are evicted, including:
  - Namespace ID
  - Workflow ID  
  - Run ID
  - Workflow state and status

## Example Configuration

To enable state-based cache eviction in your Temporal configuration:

```yaml
# config/development.yaml
dynamicconfig:
  history.cacheStateBasedEviction:
    - value: true
```

Or via dynamic configuration service:

```bash
temporal operator search-attribute create \
  --name history.cacheStateBasedEviction \
  --type Bool \
  --value true
```

## Performance Considerations

- **Enable for High-Throughput Clusters**: Most beneficial in environments with many short-lived workflows
- **Monitor Cache Hit Rates**: Ensure cache hit rates remain acceptable for your workload
- **Gradual Rollout**: Consider enabling on a subset of namespaces initially to observe impact

## Implementation Details

The feature is implemented in the workflow cache layer (`service/history/workflow/cache/cache.go`) and integrates with the existing cache release mechanism without requiring changes to the broader caching infrastructure. 