setHook(
  "rstudio.sessionInit",
  function(newSession) {
    if (newSession && is.null(rstudioapi::getActiveProject())) {
      rstudioapi::openProject(here::here())
    }
  },
  action = "append"
)

cmdstanr::set_cmdstan_path("/usr/share/.cmdstan")
