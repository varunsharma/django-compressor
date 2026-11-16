CI test replay
==============

The test settings use ``ci_replay.ReplayDiscoverRunner`` to run the existing
Django suite with the previous failed test first, followed by interrupted
tests, matching test modules and conventionally paired source modules affected
by changed files, historically risky tests, and the remaining suite. The
changed-file match is a bounded filename heuristic; it cannot infer arbitrary
runtime dependencies. The runner stops after the first failure. Every existing
tox environment remains represented in the Travis job list, and each is
placed in its own ordered stage so a failed job prevents later environments
from starting. The Python/Django environment list is unchanged.

History is a best-effort JSON cache in ``.ci-replay-cache``. Its stable cache
key is scoped by repository, branch, pull request, head ref, tox lane, and
Python runtime. Each unresolved test or collection failure separately retains
its originating repository, pull request, head/base refs and SHAs, commit, run
ID/attempt, job, tox lane, and Python runtime until a clean pass; the bounded
recent-context list is supplemental. Provenance is not part of the cache key.
The cache contains test identifiers and bounded risk counts, never test output.

Travis restores caches for pull-request builds but only stores cache changes
from normal pushes, so a new pull-request failure may not persist to the next
PR run. Concurrent runs sharing a stable key may also restore the same
snapshot and the last saved snapshot can omit updates from another run. A
missing, stale, unreadable, malformed, oversized, or incompatible cache starts
with the broad candidate set, and execution still stops at the first failure.
History is best-effort, not durable state. PR, base SHA, or run-attempt fields
may be empty when Travis does not provide them. See
https://docs.travis-ci.com/user/caching/ for cache write rules.

The runner requires one Django test process and rejects parallel test
execution. The ordered singleton Travis stages provide build-wide fail-fast:
Travis cancels later stages after a failed stage, while normal cleanup for the
failing job still runs. The former non-blocking Django-master rows are now
required stages so their test failures also stop the build.

Focused adapter checks can be run with:

.. code-block:: sh

    python -m unittest discover -s ci_replay_tests -v

The repository's Travis CI remains responsible for the Python 2.6/2.7 and
Python 3.2/3.3 tox matrix. Those interpreters are not available in the local
validation environment.
