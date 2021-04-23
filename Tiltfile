# -*- mode: Python -*-

# Settings and defaults.
settings = {
  'kind_cluster_name': 'kind',
  'live_reload': True,
}

settings.update(read_json('tilt_option.json', default={}))

default_registry(settings.get('default_registry'))

# Only use explicitly allowed kubeconfigs as a safety measure.
allow_k8s_contexts(settings.get("allowed_contexts", "kind-" + settings.get('kind_cluster_name')))


# Building Docker image.
load('ext://restart_process', 'docker_build_with_restart')

project_name = 'nri-kubernetes-operator'

if settings.get('live_reload'):
  # Building daemon binary locally.
  local_resource(project_name, 'make build', deps=[
    './main.go',
  ])

  # Use custom Dockerfile for Tilt builds, which only takes locally built daemon binary for live reloading.
  dockerfile = '''
    FROM alpine:3.13
    COPY %s /usr/local/bin/
    ENTRYPOINT ["%s"]
  ''' % (project_name, project_name)

  docker_build_with_restart(project_name, '.',
    dockerfile_contents=dockerfile,
    entrypoint=project_name,
    only=project_name,
    live_update=[
      # Copy the binary so it gets restarted.
      sync(project_name, '/usr/local/bin/%s' % project_name),
    ],
  )
else:
  docker_build(project_name, '.')

# Deploying Kubernetes resources.
k8s_yaml(helm('../helm-charts-newrelic/charts/nri-metadata-injection/', name=project_name, values='values-dev.yaml'))

# Tracking the deployment.
k8s_resource('%s-nri-metadata-injection' % project_name)
