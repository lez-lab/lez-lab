# Example: Voting with ValidityWindow

A minimal voting program that uses `ValidityWindow` to enforce a voting period.
Votes submitted outside `[start_height, end_height)` are rejected by the sequencer.

> Full example coming once the block-context branch is merged upstream.
