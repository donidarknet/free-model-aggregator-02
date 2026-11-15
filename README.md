# free-model-aggregator-02

One server, two API wires, live free models behind every answer.

`main.py` (Python 3.13, standard library only) reads the brief at
`$MAKEFUN_PROMPT_FILE`, serves OpenAI Chat Completions at
`POST /v1/chat/completions` and Anthropic Messages at `POST /v1/messages`
(JSON and server-sent events), lists the catalog at `GET /v1/models`, and
answers every prompt with a real completion from a free OpenRouter model
dialled through the grader's gateway on the trial's key. `python3 main.py
--list` prints the catalog it registers.

## Routing

Rating-first, derived from the brief at runtime — never a hardcoded table.
Candidates are the brief's free models that meet the prompt's requirements
(tools, vision, JSON mode, minimum context), ordered by `rating_milli`
descending, then by id. For this competition's catalog that means the
best JSON-capable model serves JSON-mode prompts and the best overall model
serves the rest; a different seed's ratings re-derive the same way.

## Failover

Exactly one successful provider call answers a request. On a 402, 429 or
5xx refusal the same candidate is retried once, then the next candidate
that meets the requirements is tried; 400 and 401 are never retried. A
model is dead for the rest of the trial after two consecutive refusals.
Refusals are reported on the response in the `x-makefun-upstream-failures`
header (at most 8 rows), so a recovered retry stays visible to the grader.
Total gateway calls are capped below the trial's 60-call limit.

## Determinism

No wall clock and no unseeded randomness reach any request: fixed
timestamps in listings, deterministic candidate order, and every choice a
function of the brief, the requests, and the gateway's answers, so a replay
of the trial leaves the same state export.

## Trial

The server listens on `0.0.0.0` at a free port, registers it with the env
at `$MAKEFUN_ENV_URL/register`, polls `GET $MAKEFUN_ENV_URL/work`, and
exits 0 when the env says done.

## License

MIT — see `LICENSE`.
