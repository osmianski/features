# `f6` Using Profiler

Understanding performance is important. Profiler is a major tool providing useful insights into server performance. 

{{ toc }}

## Enabling Profiler

Profiler is disabled by default as, ironically, it eats some CPU cycles itself. 

To enable profiler, add the following line to `.env` file:

    PROFILE=true
    
## Displaying Request's Performance Profile

1. In browser's `Developer Tools -> Network`, click on the request, and find the following line in `Response Headers` section: 

        performance-profile: {profile_id}    

2. Open the following URL in new tab:

        {base_url}/profiler/plain-text?id={profile_id}
        
If you see "Profile no longer exists" message:

* Try again. Profiles live for 30 minutes and then deleted. This time period is specified in `profiler_time_to_live` setting in `config/settings.php`.
* Maybe some exception has been thrown while processing HTTP request. In some cases, page may have been rendered OK, but profile was not generated due to exception. Check logs, especially `temp/log/exception.log`.   

Typical performance profile contains **sections** (e.g. `cache`) and **counters** (e.g. `app`):
 
    Timer                                                Total (ms)     Count       Avg (ms)
    ----------------------------------------------------------------------------------------
    TOTAL                                                      39.9
    ----------------------------------------------------------------------------------------
    getters                                                    13.3
    ----------------------------------------------------------------------------------------
    Osm\Ui\MenuBars\Views\MenuBar::items_                       0.1         2            0.1
    Osm\Ui\Buttons\Views\Button::iterator                       0.2         6            0.0
    ...
    ----------------------------------------------------------------------------------------
    cache                                                      10.2
    ----------------------------------------------------------------------------------------
    app                                                         2.8         1            2.8
    classes                                                     1.6         1            1.6
    ...

## Recording Profiler Stats

Wrap code you want to measure as follows:

        global $osm_profiler; /* @var Profiler $osm_profiler */

        if ($osm_profiler) $osm_profiler->start('{counter}', '{section}');
        try {
            ...
        }
        finally {
            if ($osm_profiler) $osm_profiler->stop('{counter}');
        }

**Note**. Profile counters may overlap, in this case time elapsed is added to a counter that is deeper in the call stack.   

