``` java
@CheckRights(owner = true)
    @RequestMapping(path = "/api", method = RequestMethod.GET)
    default WebAsyncTask<ResponseEntity<PersistentFileDTO>> exportDataToExcel(
            @RequestParam(required = false) String text) {

        Callable<ResponseEntity<PersistentFileDTO>> callable = () -> {
            ExportShortSearchRDTO searchRDTO = fillSearchRDTO(text, includeSubFolders, folders, onlyActive);
            PersistentFileDTO persistentFileDTO = getExportExecutorService().exportData(searchRDTO);
            return ResponseEntity
                    .ok(persistentFileDTO);
        };

        ConcurrentTaskExecutor executor = new ConcurrentTaskExecutor(
                Executors.newFixedThreadPool(1));

        return new WebAsyncTask<>(Long.MAX_VALUE, executor, callable);
    }
```
