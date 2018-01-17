## P1
BOOST_THROW_EXCEPTION
```bash
3014624ms thread-0   main.cpp:35 main]
Throw location unknown (consider using BOOST_THROW_EXCEPTION)
Dynamic exception type: boost::exception_detail::clone_impl<boost::exception_detail::error_info_injector<boost::program_options::multiple_occurrences> >
std::exception::what: option 'enable-stale-production' cannot be specified more than once

```
solution:
I had two lines
```bash
enable-stale-production = false
enable-stale-production = true
```
I delete first line and all ok
