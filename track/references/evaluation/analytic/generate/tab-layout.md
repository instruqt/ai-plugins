# Tab Layout

Evaluates tab configuration in generated tracks. Tabs define how learners interact with the track environment -- terminal access, web service UIs, editor views. This rubric evaluates whether the tab YAML configuration matches the challenge's instructional needs.

## Tabs match needs

Evaluate whether challenges have all required tabs for their tasks (e.g., CLI tasks need a terminal tab, web UI tasks need a service tab, file editing tasks need an editor tab).

Score 4: All appropriate tabs are present, including supplementary tabs that aid the learning experience.
Score 5: Tabs are precisely tailored per challenge. Each challenge has exactly the tabs needed -- no more, no less.
Score 3: All minimum required tabs are present but some useful supplementary tabs are missing.
Score 2: One or two challenges lack a required tab, forcing learners to work around the limitation.
Score 1: Multiple challenges are missing required tabs for their tasks.

## Tab titles descriptive

Evaluate whether tab titles clearly communicate what the tab provides. Learners should know what they will see before clicking.

Score 4: Tab titles are descriptive and consistent. Terminal tabs name the host, service tabs name the application. Titles follow a uniform style.
Score 5: Tab titles are precise, concise, and form a clear navigation system. A learner can find the right tab immediately without trial and error.
Score 3: Tab titles are present but some are generic ("Tab 1", "Terminal") when a more descriptive name would help.
Score 2: Most tabs have generic or unclear titles. The learner must click each tab to discover its purpose.
Score 1: Tabs have no titles or misleading titles that do not match their content.

## Service tab port configuration

Evaluate whether service tabs reference the correct ports and paths for the services they expose.

Score 4: Every service tab points to the correct port and path. The service is accessible when the tab is opened. HTTPS settings are correct.
Score 5: Service tabs are precisely configured with correct ports, paths, and protocol settings. Tabs that depend on services started during the challenge have appropriate documentation or setup script coordination.
Score 3: Service tabs reference correct ports but paths or protocol settings are slightly off, requiring the learner to adjust.
Score 2: One or more service tabs point to wrong ports or have incorrect protocol settings.
Score 1: Service tabs are misconfigured and do not connect to any running service.

## Terminal tab configuration

Evaluate terminal tab configuration: correct target host, appropriate working directory, and sensible shell settings.

Score 4: Well-configured. Learners land in the right directory on the right host for the task at hand.
Score 5: Precisely configured -- correct host, optimal working directory for each challenge's tasks, appropriate shell environment. Working directories change per challenge when the task context changes.
Score 3: Correct host with a reasonable default working directory.
Score 2: Correct host but working directory is unhelpful for the challenge's tasks.
Score 1: Terminal tab points to the wrong host or is not configured.
