## Swing, threads and code structure

In a [previous blog post](https://geertjan-kuip.github.io/2025/01/28/creating-a-logger.html) I told about the log feature of my [pet project](https://github.com/GeertJan-Kuip/spaghetti). I had distributed a logger and reserved one of the 4 subscreens in the UI for the output of the log.

Unfortunately, the logger only started to output its lines after all the loading had been done. What I had wanted was a log that informed the user of the success or failure of each loading phase. And while loading, Swing froze, although that was not very noticable because loading took less than a second. 

The reason for this malfunction has to do with the AWT Event Dispatch Thread (EDT), that is used for all tasks concerning the updating of the Swing user interface. Without explicit action, all code in the program will be executed on this thread which means that the Swing interface freezes once a large task is added to the EDT. The solution is to use SwingUtilities.InvokeLater() and SwingWorker. The former puts code on the EDT, the latter puts it on another background thread.

Last week I implemented this and I had noticed that in most of the examples I found, the SwingWorker interface was most often implemented as an anonymous class within the class where it was used. The alternative option, creating a separate class to do all the background work (with SwingWorker implemented), was less often used. Nevertheless I decided to go for the latter option, and created a package named 'swingworkers' in which I put three classes that implemented SwingWorker. Here all the heavy, time-consuming stuff should be done, like below:

```
public class WorkerClassData extends SwingWorker<String, String>{

    ActivityLogger logger;

    public WorkerClassData(ActivityLogger logger) {

        this.logger = logger;
    }

    @Override
    protected String doInBackground() throws Exception {

        SQLiteReader sqlReader = new SQLiteReader(logger);

        try {
            sqlReader.getClasses();
        } catch (SQLException e) {
            throw new RuntimeException(e);
        }

        try {
            sqlReader.getClassScopes();
        } catch (SQLException e) {
            throw new RuntimeException(e);
        }

        ArrayList<ClassContainer> classList = sqlReader.getClassList();

        SQLiteWriter sqlWriter = new SQLiteWriter(logger);

        try {
            sqlWriter.writeClassTable(classList);
        } catch (SQLException e) {
            throw new RuntimeException(e);
        }

        try {
            sqlReader.getClassRelations();
        } catch (SQLException e) {
            throw new RuntimeException(e);
        }

        ArrayList<Integer> classRelations = sqlReader.getClassRelationsData();

        try {
            sqlWriter.writeClassRelationsTable(classRelations);
        } catch (SQLException e) {
            throw new RuntimeException(e);
        }

        return "";
    }

    protected void done(){

        logger.logAction("Load process completed");
    }

}
```

While I was initially happy with the orderliness of my solution, I soon started to realize that having those separate SwingWorker classes required a lot of variables to be passed to them. The Controller class that created the SwingWorkers was meant to be the central hub of the program but by delegating important work to the SwingWorkers this pattern broke. I had let the SwingWorkers make calls to the SQLite classes where reading and writing of the database was done, even though I had decided earlier that those calls should only be made by the Controller instance. 

So, as my folder/package structure became clearer, the complexity of the program increased and actually it wasn't worth it. Using anonymous classes would have been a better option here. I might reshuffle the code in the future and have anonymous classes within my Controller class to execute all the code that would normally clutter the EDT. 

As far as the logging is concerned, I put log statements in small blocks like the one below. It seems to work perfectly.

```
            SwingUtilities.invokeLater(new Runnable() {
                public void run(){
                    logger.logAction("PROBLEM: cannot close SQLite database");
                }
            });
```