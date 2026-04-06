# FAQ

## Why does the notebook use GPS altitude for flight detection?

Because `ALT:Altitude` is a LIDAR height/range signal. It can saturate and it can drop near zero when blocked, so it is not a reliable flight-state signal.

## Why are some `Elapsed` values unusable in CSV files?

Some CSV files were exported through Excel in a way that collapses `Elapsed` into repeated scientific-notation values. The notebook rebuilds elapsed time from `Time` when that happens.

## Why are `ground_*` fields blank on some rows?

That is expected when:

- the row is `On Ground`
- required inputs are missing
- the LIDAR is blocked
- the LIDAR is saturated
- the rotated beam does not point sufficiently downward

## Why are CSV `Names of Flights` rows empty in flight?

Because CSV files do not natively carry the segment-marker column used in XLSX files, and the current policy is to keep unlabeled CSV in-flight rows empty.

## Why should I not trust the checked-in `-results.csv` files blindly?

Because documentation and code can evolve faster than committed result snapshots. Regenerate outputs from the current notebook when you need authoritative examples.

## Can I use this in Colab?

Yes. Use the Colab link on the home page, but remember to upload or mount your own data if you are not using the example repository files.
