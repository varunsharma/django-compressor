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
incompatible cache starts a cold run, which still executes the full suite.
History retention depends on the CI provider preserving or restoring this
directory; Travis jobs should be treated as cold runs unless an external cache
is configured. Cache history is not a durable record of test outcomes.

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
