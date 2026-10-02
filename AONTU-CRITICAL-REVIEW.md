# Aontu: Useful Constraints, Difficult Troubleshooting

My first encounter with Aontu came through an unsuccessful attempt to generate a REST Countries SDK on Windows. The Voxgig SDK generator stopped during setup, and later commands produced errors involving `.aontu` files. That experience gave me a specific question: how easy is it to understand and repair a model when the surrounding toolchain fails?

This is a review from that perspective. I explored Aontu’s documented modelling approach and compared it with the diagnostics I encountered. I have not independently established the cause of the integration failure, and I would not attribute every generator problem to the language underneath it.

Aontu combines data, constraints, and defaults through unification. Compatible statements narrow the possible result; conflicting statements produce an error. Statement order does not provide the override behaviour developers may expect from ordinary configuration merging. That distinction is central to understanding the language.

Consider this example, following the defaults syntax in Aontu’s documentation:

```aontu
port: *8080 | integer
port: 9090
```

The result is:

```json
{"port": 9090}
```

The preferred value is `8080`, but another integer is allowed. Replacing `9090` with `1.5` violates the integer constraint. This makes a useful distinction between choosing a default and accepting any replacement value. The documentation demonstrates both cases.

For me, that is a convincing reason to use Aontu. A configuration value often needs both a fallback and a rule. Keeping those together makes the intended behaviour easier to inspect. However, the syntax carries a learning cost: a newcomer must understand what the preference marker and alternative mean before confidently editing the model.

I would introduce that cost through examples showing a successful value, an allowed replacement, and a rejected replacement. A developer can understand the practical benefit before learning the terminology. Aontu’s example guide already takes this incremental approach, moving from familiar JSON into types, defaults, and shared constraints.

One particularly useful example applies the same requirements across named services:

```aontu
services: {
  &: { port: integer, replicas: *1 | integer }
  auth: { port: 8080 }
  db: { port: 5432, replicas: 3 }
}
```

Here, the shared template contributes constraints and defaults to each entry. The example guide documents this behaviour. I like the way it keeps a common rule beside the records it governs, instead of requiring repeated declarations that can drift apart. The trade-off is that understanding one service requires reading the shared template too.

That trade-off becomes important when models grow across files. Reuse reduces duplication, but a developer investigating a value needs to find every relevant contribution. Aontu’s documented ability to explain which statements contributed to a value is therefore particularly relevant. I would evaluate that explanation workflow alongside basic validation when deciding whether to adopt the language.

My own difficulty appeared earlier, while loading those files.

The initial SDK scaffold failed with `spawn npm ENOENT`. Running `npm install` manually inside the generated `.sdk` directory succeeded, but the project still did not complete generation. After several troubleshooting attempts and manual edits, a later run processed the OpenAPI definition and reached the final test-model step.

That step encountered this include:

```aontu
@"struct/test.aontu"
```

The diagnostic reported:

```text
[aontu/multisource_not_found]: source not found: struct/test.aontu
```

There were useful details in the output. It identified the including file, highlighted the directive, provided an error code, and listed attempted search paths. Those details gave me concrete material to include in a report.

Nevertheless, I struggled to turn that information into a recovery decision. Should I change the working directory, repair the generated configuration, or rerun an earlier setup step? The diagnostic showed where resolution had failed, but I could not confidently determine which part of the toolchain needed attention.

In the repository state I examined later, `.sdk/test/struct/test.aontu ` was present. This does not prove that the file was present in the same state when the error occurred, nor does it establish a resolver bug. It does show why describing the problem simply as “a missing file” would be too confident. The message establishes a failed lookup; the underlying reason needs further investigation.

There is another limitation to my evidence: I manually added model files while troubleshooting. Those changes mean the later errors cannot all be treated as observations from an untouched generated project. A useful review should acknowledge that instead of presenting every subsequent failure as a separate product defect.

My criticism is therefore about troubleshooting clarity at the boundary between Aontu and its host tool. The language can report an unresolved source, while the generator knows which setup stage should have supplied or configured it. A beginner experiences both as one workflow.

I would improve that workflow by making the resolution context prominent: the including file, the working directory, and the configured search roots. The existing search-path list contains valuable evidence, but a short explanation of the applicable lookup rules would make it easier to interpret.

At the generator level, I would also distinguish an incomplete scaffold from an invalid model before starting another build. That would have helped me understand whether editing model files was an appropriate next step. This recommendation belongs partly to the integration, rather than being a demand that Aontu understand every application using it.

I would still consider Aontu for a project where several tools need to share explicit constraints and defaults. Its examples demonstrate a practical benefit. For a small configuration file with few relationships, however, I would want a clear reason to introduce another language.

My adoption test would include deliberately broken models and unresolved includes, alongside successful generation. During my SDK attempt, the hardest part was deciding how to recover. Aontu’s diagnostics supplied evidence; the surrounding workflow needed to help me turn that evidence into the next correct action.
