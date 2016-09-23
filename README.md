# PDLx::Algorithm::Center

Various methods of finding the center of a sample

# FUNCTIONS

## sigma\_clip

Center a dataset by iteratively excluding data outside of a radius equal to a specified number of standard deviations

Elements may be individually weighted.

Usage:

    $results = sigma_clip( coords => $coords,
                           weight => $weight,
                           mask   => $mask,
                           %opts);

    $results = sigma_clip( weight => $img, %opts);

**sigma\_clip** finds the center of a data set by:

1. ignoring the data whose distance to the current center is a specified
number of standard deviations
2. calculating a new center by performing a (weighted) centroid of the
remaining data
3. calculating the standard deviation of the distance from the data to
the center
4. repeat at step 1 until either a convergence tolerance has been met or
the iteration limit has been exceeded

The initial center may be explicitly specified, or may be calculated
by performing a (weighted) centroid of the data.

The initial standard deviation is calculated using the initial center and either
the entire dataset, or from a clipped region about the initial center.

### Options

The following (case-insensitive) options are available:

- `coords` => _arrayref|piddle_

    _Optional_ (see `weight`)

    The coordinates to center.  `coords` may be either a _N_ element
    list of piddles of shape _M_ or a single piddle of shape _NxM_, where

    - _N_ is the number of dimensions in the data
    - _M_ is the number of data elements

    `coords` is useful if the data are sparse; for dense data, use `weight` instead.

    `weight` may be specified with coords for sparse, weighted data.

- `weight` => _piddle_

    _Optional_ (see `coords`)

    Data weights.

    For sparse data (i.e., when used with `coords`)
    `weight` must be a piddle of shape _M_, where _M_ is the number of
    data elements in `coords`.

    For densely packed data, `weight` is a piddle of shape _NxM_, where

    - _N_ is the number of dimensions in the data
    - _M_ is the number of data elements

- `mask` = _piddle_

    _Optional_

    This specifies data elements to ignore completely.
    True values indicate elements to be used, false those to be ignored.

    For sparse data (i.e., when used with `coords`) `mask` must be a
    piddle of shape _M_, where _M_ is the number of data elements in
    `coords`.

    For densely packed data, `mask` should have the same shape as `weight`.

- `nsigma` => _float_

    _Required_

    The number of standard deviations at which to clip.

- clip => _scalar|arrayref_

    _Optional_

    The clipping radius used to determine the initial standard deviation.

- `center|centre` => _arrayref_|_1D piddle_

    _Optional_

    The initial center.  Defaults to the (weighted) average of the data.

- `is_converged` => _subroutine reference_

    _Optional_

    A subroutine which determines whether the iterations have converged.
    It is called with two iteration objects which contain information about the
    previous and current iterations, i.e.

        $stop_iteration = is_converged( $last, $current );

    The structure of the objects is described in ["Iteration Results"](#iteration-results).

    It should return true if convergence has been achieved, false
    otherwise.

    The `is_converged` routine is passed references to the **actual**
    objects used by **sigma\_clip** to keep track of the iterations.  This
    means that the `is_converged` routine may manipulate the starting
    point for the next iteration by altering its `$current` parameter.

    The default behavior is to stop if both the standard deviation and
    center have not changed between iterations, or if the `dtol` option
    was specified, the centers are closer than `dtol`.

- `iterlim` => _integer_

    _Optional_

    The maximum number of iterations to run.  Defaults to 10.

- `dtol` => _float_

    _Optional_

    If specified, and the default convergence behavior is in use (see the
    `is_converged` option) iteration will cease when successive centers are
    closer than the specified distance.

- `log` => _subroutine reference_

    _Optional_

    A subroutine which will be called at the end of each iteration. It is passed
    a copy of the current iteration's results object see ["Iteration Results"](#iteration-results)).

### Iteration Results

The results for each iteration are stored in object of class
`PDLx::Algorithm::Center::sigma_clip::Iteration` with the
following attributes/methods:

- `center` => _piddle|undef_

    A 1D piddle containing the derived center.  The value for the last
    iteration will be undefined if all of the elements have been clipped.

- `iter` => _integer_

    The iteration index.  An index of `0` indicates the values determined
    before the iterative loop was entered, and reflects the initial
    clipping and mask exclusion.

- `nelem` => _integer_

    The number of data elements used in the center.

- `weight` => _float_

    The combined weight of the data elements used to determine the center.

- `sigma` => _float|undef_

    The standard deviation of the data.
    The value for the last iteration will be
    undefined if all of the elements have been clipped.

- `variance` => _float|undef_

    The calculated variance (i.e., `sqrt( sigma )`.
    The value for the last iteration will be
    undefined if all of the elements have been clipped.

- `clip` => _float|undef_

    The clipping radius.  This will be undefined for the first iteration
    if the `clip` option was not specified.

- `dist` => _float_

    _Optional_

    The distance between the previous and current centers. This is present
    only if the default convergence routine is in use.

### Returned Results

**sigma\_clip** returns an object of class
`PDLx::Algorithm::Center::sigma_clip::Result`.  It is a subclass
of `PDLx::Algorithm::Center::sigma_clip::Iteration` (the common
attributes refer to the results of the final iteration) with these
additional attributes/methods:

- `iterations` => _arrayref_

    A list of the iteration result objects.

- `success` => _boolean_

    True if the iteration converged, false otherwise.

- `error` => _error object_

    If convergence has failed, this will contain an error object
    describing the failure.  See ["Errors"](#errors).

### Errors

Errors are represented as objects in the following classes:

- Parameter Validation

    These are unconditionally thrown.

        PDLx::Algorithm::Center::parameter
        PDLx::Algorithm::Center::parameter::type
        PDLx::Algorithm::Center::parameter::dimension
        PDLx::Algorithm::Center::parameter::missing
        PDLx::Algorithm::Center::parameter::value

- Iteration

    These are stored in the result object's `error` attribute.

        PDLx::Algorithm::Center::iteration::limit_reached
        PDLx::Algorithm::Center::iteration::empty

The objects stringify to a failure message.

# AUTHOR

Diab Jerius, <djerius@cpan.org>

# COPYRIGHT AND LICENSE

Copyright 2016 Smithsonian Astrophysical Observatory

This software is released under the GNU General Public License.  You
may find a copy at

          http://www.gnu.org/licenses
