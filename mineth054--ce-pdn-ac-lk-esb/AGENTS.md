# CE-PDN-AC-LK-ESB — AI Agent Guide

- **Project shape**: WSO2 Micro Integrator integration project; Maven builds a Carbon Application (.car) containing Synapse configs under `src/main/wso2mi/artifacts/` and supporting resources under `src/main/wso2mi/resources/`.
- **Main API**: `ResearchGroup-Vision` Synapse API (`artifacts/apis/ResearchGroup-Vision.xml`) exposes `GET /research-groups/v1/vision/` and `/publications`.
- **Flow: group details**: `GET /` → call endpoint `rg-computer-vision` (https://portal.ce.pdn.ac.lk/api/taxonomy/v1/term/computer-vision) → datamapper `rg-details-data-mapper` (`resources:datamapper/rg-details-data-mapper/*.dmc`); mapper TS currently stubbed (mapFunction returns `{}`) so fill outputs when extending.
- **Flow: publications**: `GET /publications` → call endpoint `publications-filter-by-research-group` (https://api.ce.pdn.ac.lk/publications/v1/filter/research-groups/) → datamapper `rg-vision-publications-mapper` → log → respond; mapper returns `{ data: publications, metadata.count, response: "success" }` (see `rg-vision-publications-mapper.ts`).
- **Endpoints**: Defined in `artifacts/endpoints/*.xml`; referenced by key from API definitions—keep names in sync when adding/changing resources.
- **Data mappers**: TS helpers live in `resources/datamapper/**/`; compiled artifacts/schemas are registered via `resources/artifact.xml` to MI registry paths `/_system/governance/mi-resources/datamapper/...`. Update this file when adding new mappers or schemas, and reference them as `resources:datamapper/<name>/<file>` inside Synapse configs.
- **Config inputs**: `resources/conf/config.properties` is currently empty but reserved for mapper/property values; `deployment/deployment.toml` holds MI runtime config (keystore/truststore defaults to `wso2carbon` passwords). Avoid hardcoding secrets; prefer config/secret scopes via `DM_PROPERTIES` in mappers if needed.
- **Build (local CAR)**: `./mvnw clean package` (default profile) → CAR in `target/${artifactId}_${version}.car`; dependencies copied to `deployment/libs` (transitives excluded for Synapse/Axis2). Java source/target 1.8.
- **Build (Docker)**: `./mvnw -Pdocker clean package` → runs config mapper, builds CAR, stages under `target/tmp_docker`, then `io.fabric8:docker-maven-plugin` builds image `${artifactId}:${version}` using base `${dockerfile.base.image}` (default `wso2/wso2mi:4.5.0`). Requires Docker daemon.
- **Docker image contents**: `deployment/docker/Dockerfile` copies `CompositeApps/*.car` into `${WSO2_SERVER_HOME}/repository/deployment/server/carbonapps/` and replaces keystore/truststore from `deployment/docker/resources/`. Add custom libs by uncommenting `COPY libs/*.jar`.
- **Tests**: Uses `org.wso2.maven:synapse-unit-test-maven-plugin` bound to `test`. Invoke with `./mvnw test -DtestFile=<case.xml> -Dmaven.test.skip=false`; test cases expected under `src/test/wso2mi/`. Override MI server settings via `-Dtest.server.host/port/path/version` or set properties in the active profile.
- **Profiles**: `default` (active) and `docker`. Both set `server.*` props from `test.server.*`. Docker profile shifts CAR build to `generate-sources` phase and runs config mapper plus docker image build.
- **Dependencies**: Maven repos are WSO2 Nexus; avoid adding public deps already bundled in MI (`org.apache.synapse`, `org.apache.axis2` are excluded). Keep additional runtime jars in `deployment/libs`.
- **Deployment without Docker**: Copy built CAR from `target/*.car` into MI `repository/deployment/server/carbonapps/`; ensure matching keystore/truststore if you rely on defaults (`wso2carbon` password).
- **Conventions**: Place new Synapse artifacts by type under `src/main/wso2mi/artifacts/<type>/`; update API context/uri-templates carefully (trailing slash in `context="research-groups/v1/vision/"`). Keep resource keys stable to avoid broken references.
- **External integrations**: Calls outward to CE PDN APIs (taxonomy and publications). No auth is configured; if adding auth, wire headers in the API `inSequence` before `call`.
- **Quick refs**: API def `resources/api-definitions/ResearchGroup-Vision.yaml`; Synapse API `artifacts/apis/ResearchGroup-Vision.xml`; endpoints `artifacts/endpoints/*.xml`; datamapper TS `resources/datamapper/**`; registry mapping `resources/artifact.xml`; runtime config `deployment/deployment.toml`; Docker build `deployment/docker/Dockerfile`.

If any section is unclear or missing (e.g., test case locations or mapper expectations), tell me and I’ll refine it.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Mineth054)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/Mineth054)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
