# Notes Content Patterns

Evaluates whether notes slide content follows effective patterns for different challenge positions and track contexts, creating a polished and professional learning experience.

## Criteria

| Score | Level | Description |
|-------|-------|-------------|
| 1 | Poor | No content patterns; notes are either absent or generic placeholder text across all challenges |
| 2 | Below Standard | Notes exist but use the same pattern for every challenge regardless of position or context |
| 3 | Adequate | Some variation in notes content by challenge position, but patterns are inconsistent or underdeveloped |
| 4 | Good | Notes content is relevant and concise; welcome/splash for first challenge, context-setting for subsequent challenges (production baseline) |
| 5 | Excellent | Notes create a polished, professional learning experience -- branded styling, customer-specific elements on demo tracks, iframe embedding for rich content |

## Guidance

Different challenge positions and track contexts call for different notes content patterns.

### Welcome/splash pattern (first challenge)

The first challenge sets the tone. Use a branded splash image followed by a brief welcome:

```yaml
notes:
  - type: image
    url: https://cdn.example.com/acme-lab-splash.png
  - type: text
    contents: |
      # Welcome to Acme Platform

      In this hands-on lab, you will deploy and configure
      the Acme Platform on Kubernetes.

      No prior Kubernetes experience is required.
```

### Context-setting pattern (subsequent challenges)

Subsequent challenges need lighter notes that bridge from what was accomplished to what comes next:

```yaml
notes:
  - type: text
    contents: |
      # Configuring Observability

      Your application is deployed and handling traffic.
      Now it is time to add monitoring so you can see
      what is happening under the hood.
```

### Customer-specific dedication (SE demo tracks)

For tracks built for specific customers during sales engineering engagements, add a dedication slide:

```yaml
notes:
  - type: text
    contents: |
      <style>
      .dedication {
        text-align: center;
        padding: 40px;
        font-size: 1.2em;
      }
      .dedication img {
        max-width: 200px;
        margin-bottom: 20px;
      }
      </style>

      <div class="dedication">

      ![Acme Corp](https://cdn.example.com/acme-logo.png)

      Built for the **Acme Corp** platform team

      </div>
```

### Branded styling with inline CSS

Inline CSS in text slides allows branded colors, fonts, and layout:

```yaml
notes:
  - type: text
    contents: |
      <style>
      h1 { color: #1a73e8; border-bottom: 3px solid #1a73e8; }
      h2 { color: #5f6368; }
      body { font-family: 'Google Sans', sans-serif; }
      </style>

      # Google Cloud Lab

      ## Infrastructure as Code with Terraform

      Provision cloud resources using declarative configuration.
```

### Iframe embedding for external walkthroughs

For rich external content that goes beyond what markdown can express:

```yaml
notes:
  - type: text
    contents: |
      <style>
      iframe {
        width: 100%;
        height: 500px;
        border: none;
        border-radius: 8px;
      }
      </style>

      <iframe src="https://app.example.com/walkthrough/intro"></iframe>
```

Good -- first challenge with splash and concise welcome:

```yaml
notes:
  - type: image
    url: https://cdn.example.com/hashicorp-vault-splash.png
  - type: text
    contents: |
      # Secrets Management with Vault

      Learn to store, access, and manage secrets
      using HashiCorp Vault.
```

Good -- subsequent challenge with forward-looking context:

```yaml
notes:
  - type: text
    contents: |
      # Dynamic Secrets

      Static secrets work, but they require manual rotation.
      Vault can generate short-lived credentials on demand --
      let's see how.
```

Good -- SE demo track with customer dedication:

```yaml
notes:
  - type: text
    contents: |
      <style>
      h1 { text-align: center; margin-top: 60px; }
      p { text-align: center; color: #666; }
      </style>

      # Prepared for MegaCorp Engineering

      A hands-on introduction to our platform,
      tailored to your infrastructure.
```

Bad -- same generic intro on every challenge:

```yaml
# Challenge 1, 2, 3, 4 all have:
notes:
  - type: text
    contents: |
      # Welcome

      This is the next challenge. Let's get started!
```

Bad -- notes that teach instead of tease:

```yaml
notes:
  - type: text
    contents: |
      # Kubernetes Networking

      Kubernetes networking follows four rules:
      1. Every pod gets its own IP address
      2. Pods on any node can communicate with pods on any other node
      3. Agents on a node can communicate with all pods on that node
      4. [... three more paragraphs of CNI explanation ...]
      # This is a lesson, not a loading screen note
```

Bad -- broken iframe from HTTP source:

```yaml
notes:
  - type: text
    contents: |
      <iframe src="http://insecure-site.com/demo"></iframe>
```

## What to Watch For

- First challenge notes set the tone for the entire track -- invest in a polished splash image and concise welcome text
- Subsequent challenge notes should be lighter -- one text slide bridging from previous context to upcoming work
- Customer dedication slides on SE demo tracks show professionalism and make the demo feel tailored
- Inline CSS is powerful but keep it simple -- complex layouts may render differently across screen sizes
- Iframe sources must be HTTPS and must allow embedding (no X-Frame-Options DENY on the source)
- Do not use notes as a teaching medium -- they are a loading screen; save instruction for the assignment
- Branded colors and fonts should match the customer or product being demonstrated
- Each slide should be readable in 3-5 seconds since slides auto-advance during loading
