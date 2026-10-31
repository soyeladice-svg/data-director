# Writing an agent

Each agent is one package and one A2A service under `agents/<name>/`. Agents share the contract
and helpers in `sdk/`, and are governed by the harness in `harness/`. An agent must not import
the workbench, the app or another agent.

See [`agents/README.md`](../../../agents/README.md) for the layout, the alpha version convention,
and the complete recipe for an agent. The existing `example.hello` template walks that recipe.

## Follow the `example.hello` agent

1. **Own the input and output classes.** Start with the LinkML definition in
   [`hello.yaml`](../../../agents/example-hello/src/dd_agent_example_hello/schema/hello.yaml).
   `Salutation` has `greeted_name` and an optional `language`; `Greeting` contains
   `greeting_text`, its `greeting_derivation`, and grounding references. The matching Pydantic
   models live in [`classes.py`](../../../agents/example-hello/src/dd_agent_example_hello/classes.py).
   Regenerate the committed schemas when the LinkML definitions change, as the schema's header
   instructs. The agent's classes are not added to the shared core.
2. **Declare the interface.** In
   [`agent.py`](../../../agents/example-hello/src/dd_agent_example_hello/agent.py),
   `HelloWorld.spec` names the `example.hello` agent, accepts `Salutation`, returns `Greeting`,
   and sets `grounding_mode` to `none`. It declares that `greeting_text` was derived from a
   template and records that derivation in `greeting_derivation`.
3. **Implement `run(request, ctx)`.** The conductor validates the declared input class before
   calling `run`. The agent reads `request.input`, chooses a deterministic greeting, and returns
   an `AgentResult`. For an unsupported language it returns `abstained` with
   `capability-not-implemented` rather than raising an exception or guessing. The returned
   `Greeting` cites the input via `ctx.input_ref` and `ctx.input_hash`; the agent supplies an
   `EvidenceItem`, not fabricated identifiers or provenance.
4. **Serve the agent independently.** The package exports a factory and uses `dd_sdk.serve`
   to expose its A2A endpoint. The harness calls it as a separate service, not by importing
   the agent into the conductor. Consult the service commands and sample registration in
   [`agents/README.md`](../../../agents/README.md).
5. **Test the contract.** The committed
   [`test_hello.py`](../../../agents/example-hello/tests/test_hello.py) asserts that the
   `world` sample produces `Hello, world!`, the payload cites the input hash, and no model
   was used. It also tests an unknown language, an undeclared input class, the manifest,
   and the agent-owned schema. Run the relevant tests from the repository root using the
   setup instructions in [Setting up](setup.md), then run the documented repository checks.

The important separation is that **the agent builds its payload and evidence while the harness
checks its declared contract and records the run**. The template's greeting has no external
lookup or model call; copying it does not by itself make a production-ready agent.
