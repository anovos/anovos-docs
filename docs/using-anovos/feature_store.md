# Feature Store Integration

Feature stores are an essential building block of a modern MLOps setup.
For an introduction into the concept and an overview of available options and vendors, see the
[Feature Store Comparison & Evaluation](https://mlops.community/learn/feature-store/)
on the [MLOps Community website](https://mlops.community/).

_Anovos_ provides integration with [Feast](https://www.feast.dev), a widely used open source feature store,
out of the box.
Using the same abstractions, it is straightforward to integrate _Anovos_ with other feature stores.

If there is a particular feature store integration you'd like to see supported by _Anovos_,
[let us know!](../community/communication.md)

## Using _Anovos_ with Feast

### Prerequisites

```bash
feast init
```

### Adding Feast export to your _Anovos_ workflow

Copy the following template into your workflow configuration file:

```yaml
write_feast_features:
  file_path:
  entity:
  id_col:
  timestamp_col:
  create_timestamp_col:
  source_description:
  entity_description:
  view_name:
  view_owner:
  view_ttl_in_seconds:
  owner:
  create_timestamps: True
  file_configs:
    mode: overwrite
```

### Exporting data to Feast

## Integrating _Anovos_ with other feature stores
