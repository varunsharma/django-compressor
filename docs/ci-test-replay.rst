CI test replay
==============

The test settings use ``ci_replay.ReplayDiscoverRunner`` to run the existing
Django suite with the previous failed test first, followed by interrupted
tests, matching test modules and conventionally paired source modules affected
by changed files, historically risky tests, and the remaining suite. The
changed-file match is a bounded filename heuristic; it cannot infer arbitrary
runtime dependencies. The runner stops after the first failure. The existing
tox matrix, lint command, coverage collection, and test labels remain in place.

History is a best-effort JSON cache in ``.ci-replay-cache``. It contains test
identifiers, bounded risk counts, and CI repository, branch, pull request,
commit, run, job, lane, and Python context. It does not contain test output.
The cache is advisory: an absent, unreadable, malformed, oversized, or
incompatible cache starts with the broad candidate set, and the runner still
stops at the first failure. Travis caches this
directory between normal pushes. Travis restores caches for pull-request
builds but only stores cache modifications from normal pushes, so new failures
from a pull-request build are not guaranteed to persist to its next run. A
cold or stale cache therefore remains possible; this history is not durable.
PR, base SHA, or run-attempt values may be empty when Travis does not expose
them to a job. See https://docs.travis-ci.com/user/caching/ for cache scope
and write rules.

Unresolved test and collection failures keep their originating repository, pull request, head/base refs and SHAs, run ID/attempt, job, tox lane, and Python runtime attached per test ID. That provenance has no 12-run expiry and is removed only after a clean pass; the bounded recent-context list is supplemental. The cache schema change starts once from cold history when older cached data is present.

The runner requires one Django test process and rejects parallel test
execution. The current Travis matrix runs separate tox environments, so a
failure stops tests in its own job; Travis does not cancel other already
running matrix jobs when one job fails.

Focused adapter checks can be run with:

.. code-block:: sh

    python -m unittest discover -s ci_replay_tests -v

The repository's Travis CI remains responsible for the Python 2.6/2.7 and
Python 3.2/3.3 tox matrix. Those interpreters are not available in the local
validation environment.
