# Slack routed-identity backport — review bundle

This is an **artifact-only backport review**, not a duplicate source fix for current upstream, an installation, or a deployment. The adjacent patch changes only the older adapter's `_build_identity_prompt` and adds a synthetic regression test. It must not be applied to current main, which already contains the corrected guidance.

## Immutable identities

- Applicable public source base: `17155e3ae04d376dd8eba2e65f3dd966e67ab1ba`.
- Tested candidate source revision: `9c9f2acfe73ec65ac21b64e5b0e0c4758a4b06e3`.
- Candidate Git tree: `a4df83fcd29b42bd4c7efef4842370b9ec1181d6`.
- Patch: `slack-routed-identity-17155e3.patch`.
- Patch SHA-256: `7ca452d4adf6f15dfbd21d6d79cdbe7cb57bd01c240d070fd3d7de4d5c335dd5`.
- Frozen new test SHA-256: `9ca7c65a4a2c7415fe648a890c1c93f8d23b01cb6d8f008f07f22dda1c7b66e1`.

The local candidate commit is a source identity, not a claim that it is remotely published. Patch application/tree equality must be checked separately. No unrelated local history is included.

## Why this patch

The older native Slack handler admits an own-ID mention, strips that token, and forwards the normalized message. Its identity guidance nevertheless requires a visible own-name mention. The patch backports the guidance already present in upstream [`9cb34e9e285dadd0c6d2463e9b094052530fc365`](https://github.com/NousResearch/hermes-agent/blob/9cb34e9e285dadd0c6d2463e9b094052530fc365/plugins/platforms/slack/adapter.py): trust the adapter's routing decision and do not reject a delivered turn solely because the routing mention was stripped.

Admission, configured mention patterns, channel controls, authorization, name resolution, command stripping and intentional-silence/incomplete-output protections remain unchanged. This does not force every admitted message to receive a reply.

## Executed bounded verification

Native `scripts/run_tests.sh`, one worker, zero file retries, isolated homes and a network/write audit guard:

- Before implementation: 10 focused controls passed; 1 expected missing-guidance assertion failed. No setup/collection failure was counted as behavioral RED.
- Implementer verification on the exact candidate source/test bytes: all 11 focused cases passed.
- Unchanged preservation selector: all 382 cases passed across seven files.
- Frozen-cohort verifier: 178 records verified, with only the authorized adapter divergence permitted after implementation.

Preservation files: `test_slack_mention_humanization.py`, `test_slack_mention.py`, `test_slack.py`, `test_gateway_silence_tokens.py`, `test_incomplete_gateway_turns.py`, `test_shared_group_sender_prefix.py`, `test_discord_channel_prompts.py` under `tests/gateway/`.

These are bounded handler/context tests, not live Slack or model acceptance. The gateway handoff/name lookup are test boundaries. A generic spaced identity value exercises template/cache handling; it does not prove `auth.test.user` is a UI profile display name. Plain full/partial names are distinct from structured user-ID mentions in the tested strict/default configuration. Separately configured mention patterns are not redefined.

The current-main full same-test oracle was blocked by its runner scratch-root boundary; only the actual current-main helper's narrower 10-assertion oracle ran successfully. The Python audit guard is not an OS sandbox. Some preservation cases attempted writes which the guard denied; passing assertions do not mean zero attempted I/O. No full-repository suite, live provider cause, exact profile-name lookup or end-to-end automatic reply recovery is claimed.

## Review application

Use a **separate disposable checkout already at the public base**, never the installed runtime:

```sh
git apply --check /path/to/slack-routed-identity-17155e3.patch
git apply --index /path/to/slack-routed-identity-17155e3.patch
git write-tree
```

The resulting tree must equal the candidate tree above. The native focused selector is `scripts/run_tests.sh -j 1 --file-timeout 120 --file-retries 0 tests/gateway/test_slack_routed_identity.py -q`, with an independently provisioned compatible test environment and isolated application home. This bundle does not itself provision that environment or authorize deployment.

A legacy-base publication attempt returned GitHub's workflow-check timeout, so this carrier avoids changing credentials, scopes or workflow files. No merge, service restart, runtime change, notification-routing change or further live Slack write is part of this review.
