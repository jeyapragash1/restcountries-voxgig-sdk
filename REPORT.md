# Voxgig SDK Generator Mini Task Report

## API Selected

I selected the REST Countries API.

API documentation: https://restcountries.com/docs

Repository: https://github.com/jeyapragash1/restcountries-voxgig-sdk

## Goal

The goal was to generate an unofficial open-source SDK for selected REST Countries endpoints using the Voxgig SDK generator.

The selected endpoints were:

- GET /all
- GET /name/{name}

## Commands Used

npm create @voxgig/sdkgen restcountries
cd restcountries-sdk/.sdk
npm install
npm run generate

## Result

The Voxgig SDK project structure was created and project-level files were generated, including README, LICENSE, NOTICE, CHANGELOG, SECURITY, AGENTS and CLAUDE files.

After adding a small OpenAPI definition for REST Countries, the OpenAPI parsing step completed successfully and the SDK model generation step also completed.

However, the full SDK target folders such as ts and py were not generated successfully within the time-box. The final test model step failed because some scaffold/test files were missing.

## Main Issue Encountered

The first generator run failed on Windows with this error:

Voxgig Create SDK Error: Failed to start npm: spawn npm ENOENT

My npm was installed and available:

npm -v
11.5.1

where npm
C:\nvm4w\nodejs\npm
C:\nvm4w\nodejs\npm.cmd

This suggests the generator had difficulty spawning npm from my Windows/nvm-windows environment.

## Follow-on Issues

Because the first run failed, the generated project was incomplete. Several expected files were missing, including API, entity, feature, target, flow and edition config files.

The generated OpenAPI file was also empty, so I manually added a small OpenAPI 3.0.3 definition for the selected REST Countries endpoints.

After that, OpenAPI parsing and SDK model generation succeeded, but the final test model step failed with:

source not found: struct/test.aontu

## Developer Experience Observations

The Voxgig SDK generator appears powerful, especially because it can generate a structured SDK project from an OpenAPI definition.

The main difficulty was the first-run experience on Windows. The npm spawn error created a partially generated project, and after that the errors appeared one missing file at a time. This made the issue harder to understand.

## Suggestions

- Add a preflight check for node, npm and npm spawning before generation starts.
- If npm spawning fails, avoid leaving a partial project or provide a clear cleanup/resume command.
- Add a doctor command to detect missing scaffold files.
- If the OpenAPI file is empty, show a friendly message with a minimal example.
- Provide clearer Windows/nvm-windows troubleshooting notes.

## Time Box

I stopped after the generator setup and scaffold issues continued beyond the intended 30-minute task window. I documented the issue instead of continuing to repair generated internals manually.

## AI Usage

I used AI assistance to guide the debugging process and structure this report. I manually ran the commands, reviewed the terminal output, and documented the actual errors encountered.