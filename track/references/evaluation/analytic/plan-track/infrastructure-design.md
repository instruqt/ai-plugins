# Infrastructure Design

Evaluates config.yml resource design quality. The infrastructure must match what the track needs -- correct machine types, images, ports, and cloud account configuration. Over-provisioning wastes quota and startup time; under-provisioning causes failures mid-track.

## Resource specification

Evaluate whether config.yml defines the right resources for the track's requirements.

Score 4: Resources match track needs. Correct images and sizing for each virtual machine or container. Quota-compliant. Ports exposed for all service tabs.
Score 5: Infrastructure uses the most efficient pattern for the use case. Machine types justify their cost. Cloud accounts use least-privilege IAM. Startup time is optimized by choosing the smallest viable images.
Score 3: Resources are functional but not optimized -- over-provisioned machines, unnecessary VMs, or images larger than needed for the workload.
Score 2: Resources are present but mis-sized, use wrong images, or are missing port definitions that tabs require.
Score 1: Missing or invalid config.yml. Resources do not match what the track needs. The track cannot start or critical services are unavailable.

## Image selection

Evaluate whether container and VM images are appropriate for the technology being taught.

Score 4: Images match the technology stack. Versions are appropriate for the track content. No deprecated or end-of-life images.
Score 5: Smallest appropriate image for each resource. Specific version tags chosen to match the track's content exactly. Image choices minimize startup time and resource consumption.
Score 3: Reasonable image choices but not optimal -- using a full desktop image when a minimal server image would suffice, or version is older than necessary.
Score 2: Images work but are oversized, use unversioned tags, or include unnecessary components.
Score 1: Wrong images entirely -- an Ubuntu image for a Windows track, or images that do not include the required technology.

## Port and service exposure

Evaluate whether all services that need to be accessible via tabs or external URLs have their ports correctly exposed in config.yml.

Score 4: Every service tab has a corresponding port exposed. Port numbers match the actual service configuration. No unnecessary ports exposed.
Score 5: Ports are precisely mapped to services. Each exposed port has a clear purpose tied to a tab or check script requirement. Security-sensitive ports are not exposed unnecessarily.
Score 3: Most ports are correctly exposed but one service tab would fail to connect due to a missing or incorrect port mapping.
Score 2: Several services lack port exposure. Learners would encounter connection failures during the track.
Score 1: No ports exposed, or port definitions are entirely wrong for the services being used.

## Cloud account configuration

Evaluate whether cloud account resources (GCP, AWS, Azure) are configured with appropriate permissions and constraints.

Score 4: Cloud accounts have correct IAM roles for the track's requirements. Permissions are scoped to what the track needs. Region and quota settings are appropriate.
Score 5: Least-privilege IAM configuration. Permissions are precisely scoped to the track's needs with no excess access. Region selection optimizes for availability and cost. Quota limits are explicitly considered.
Score 3: Cloud accounts have working permissions but are broader than necessary -- admin roles where specific permissions would suffice.
Score 2: Cloud accounts are configured but permissions are too broad or too narrow, causing either security concerns or track failures.
Score 1: Cloud accounts are missing when the track requires cloud resources, or IAM configuration is fundamentally broken.
