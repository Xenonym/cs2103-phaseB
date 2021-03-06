# Xenonym
###### /java/org/graphstream/algorithm/test/WidestPathTest.java
``` java
    @Test
    public void widestPathTest() {
        Graph g = toyGraph();

        WidestPath d = new WidestPath(Dijkstra.Element.EDGE, "result", "length");
        d.init(g);
        Node source = g.getNode("A");
        d.setSource(source);

        // check the source node
        assertEquals(d.getSource(), source);

        d.compute();

        // check parent access methods
        assertNull(d.getParent(source));
        assertNull(d.getEdgeFromParent(source));
        assertEquals(source, d.getParent(g.getNode("C")));
        assertEquals(g.getEdge("CD"), d.getEdgeFromParent(g.getNode("D")));
        assertNull(d.getParent(g.getNode("G")));
        assertNull(d.getEdgeFromParent(g.getNode("G")));


        // check path widths
        assertEquals(Double.POSITIVE_INFINITY, d.getPathLength(g.getNode("A")), 0);
        assertEquals(14, d.getPathLength(g.getNode("B")), 0);
        assertEquals(9, d.getPathLength(g.getNode("C")), 0);
        assertEquals(9, d.getPathLength(g.getNode("D")), 0);
        assertEquals(9, d.getPathLength(g.getNode("E")), 0);
        assertEquals(9, d.getPathLength(g.getNode("F")), 0);
        assertEquals(Double.NEGATIVE_INFINITY, d.getPathLength(g.getNode("G")), 0);

        // check tree length
        assertEquals(53, d.getTreeLength(), 0);

        // check nodes on path A->E
        String[] nodesAe = {"E", "B", "A"};
        int i = 0;
        for (Node n : d.getPathNodes(g.getNode("E"))) {
            assertEquals(nodesAe[i], n.getId());
            i++;
        }
        assertEquals(3, i);

        // check edges on path A->F
        String[] edgesAf = {"CF", "AC"};
        i = 0;
        for (Edge e : d.getPathEdges(g.getNode("F"))) {
            assertEquals(edgesAf[i], e.getId());
            i++;
        }
        assertEquals(2, i);

        // check if path A->E is constructed correctly
        List<Node> ln = d.getPath(g.getNode("E")).getNodePath();
        assertEquals(3, ln.size());
        for (i = 0; i < 3; i++) {
            assertEquals(nodesAe[2 - i], ln.get(i).getId());
        }

        // There is no path A->G
        assertFalse(d.getPathNodesIterator(g.getNode("G")).hasNext());
        assertFalse(d.getPathEdgesIterator(g.getNode("G")).hasNext());

        d.clear();
        assertFalse(source.hasAttribute("result"));
    }
```
###### /java/seedu/address/logic/CommandHistoryTest.java
``` java
    @Test
    public void clear() {
        history.add("list");
        history.add("clear");
        history.clear();
        assertEquals(Collections.EMPTY_LIST, history.getHistory());
    }
```
###### /java/seedu/address/logic/commands/BackupCommandTest.java
``` java
public class BackupCommandTest {

    @Rule
    public TemporaryFolder testFolder = new TemporaryFolder();

    private BackupCommand backupCommand;
    private XmlAddressBookStorage addressBookStorage;

    @Before
    public void setUp() {
        try {
            backupCommand = new BackupCommand();
            addressBookStorage = new XmlAddressBookStorage(getTempFilePath("ab"));
            addressBookStorage.saveAddressBook(getTypicalAddressBook());
            JsonUserPrefsStorage userPrefsStorage = new JsonUserPrefsStorage(getTempFilePath("prefs"));

            backupCommand.setData(null, null, null, new StorageManager(addressBookStorage, userPrefsStorage));
        } catch (IOException ioe) {
            fail(ioe.getMessage());
        }
    }

    private String getTempFilePath(String filename) {
        return testFolder.getRoot().getPath() + filename;
    }

    @Test
    public void execute() throws Exception {
        CommandResult result = backupCommand.execute();
        assertEquals(BackupCommand.MESSAGE_SUCCESS, result.feedbackToUser);

        byte[] original = Files.readAllBytes(Paths.get(addressBookStorage.getAddressBookFilePath()));
        byte[] backup = Files.readAllBytes(Paths.get(addressBookStorage.getBackupAddressBookFilePath()));
        assertTrue(Arrays.equals(original, backup));
    }
}
```
###### /java/seedu/address/logic/commands/ClearHistoryCommandTest.java
``` java
public class ClearHistoryCommandTest {
    private ClearHistoryCommand clearHistoryCommand;
    private CommandHistory history;
    private UndoRedoStack undoRedoStack;

    @Before
    public void setUp() {
        history = new CommandHistory();
        undoRedoStack = new UndoRedoStack();
        clearHistoryCommand = new ClearHistoryCommand();
        clearHistoryCommand.setData(new ModelManager(), history, undoRedoStack, new StorageStub());
    }

    @Test
    public void execute() throws Exception {
        history.add("command1");
        history.add("command2");
        undoRedoStack.push(new DummyCommand());
        undoRedoStack.push(new DummyUndoableCommand());
        clearHistoryCommand.execute();
        assertEquals(Collections.emptyList(), history.getHistory());
        assertFalse(undoRedoStack.canUndo());
        assertFalse(undoRedoStack.canRedo());
    }

    class DummyCommand extends Command {
        @Override
        public CommandResult execute() {
            return new CommandResult("");
        }
    }

    class DummyUndoableCommand extends UndoableCommand {
        @Override
        public CommandResult executeUndoableCommand() {
            return new CommandResult("");
        }
    }
}
```
###### /java/seedu/address/logic/commands/ColourTagCommandTest.java
``` java
public class ColourTagCommandTest {

    private Model model = new ModelManager(getTypicalAddressBook(), new UserPrefs());

    @Test
    public void execute_changeColour_success() throws Exception {
        ColourTagCommand command = new ColourTagCommand(new Tag("test"), "red");
        command.setData(model, null, null, null);

        String expectedMessage = String.format(ColourTagCommand.MESSAGE_COLOUR_TAG_SUCCESS, "test", "red");
        ModelManager expectedModel = new ModelManager(model.getAddressBook(), new UserPrefs());
        Map<Tag, String> colours = new HashMap<>(expectedModel.getUserPrefs().getGuiSettings().getTagColours());
        colours.put(new Tag("test"), "red");
        expectedModel.getUserPrefs().getGuiSettings().setTagColours(colours);

        assertCommandSuccess(command, model, expectedMessage, expectedModel);
    }
}
```
###### /java/seedu/address/logic/commands/RelPathCommandTest.java
``` java
/**
 * Contains integration tests (interaction with the Model) and unit tests for RelPathCommand.
 */
public class RelPathCommandTest {

    private Model model;
    private Model expectedModel;
    private GraphWrapper graph;
    private RelPathCommand relPathCommand;

    @Before
    public void setUp() {
        model = new ModelManager(getTypicalAddressBookWithRelationships(), new UserPrefs());
        expectedModel = new ModelManager(model.getAddressBook(), new UserPrefs());
        graph = GraphWrapper.getInstance();
        graph.buildGraph(model);
    }

    @Test
    public void execute_hadUndirectedPath_pathFound() {
        relPathCommand = new RelPathCommand(INDEX_FIRST_PERSON_WITH_REL, INDEX_SECOND_PERSON_WITH_REL);
        relPathCommand.setData(model, null, null, null);
        String expectedMessage = String.format(MESSAGE_PATH_FOUND, JAMES.getName().fullName,
                KELVIN.getName().fullName);

        assertCommandSuccess(relPathCommand, model, expectedMessage, expectedModel);
    }

    @Test
    public void execute_hasDirectedPath_pathFound() {
        relPathCommand = new RelPathCommand(INDEX_SECOND_PERSON_WITH_REL, INDEX_THIRD_PERSON_WITH_REL);
        relPathCommand.setData(model, null, null, null);
        String expectedMessage = String.format(MESSAGE_PATH_FOUND, KELVIN.getName().fullName,
                LISA.getName().fullName);

        assertCommandSuccess(relPathCommand, model, expectedMessage, expectedModel);
    }

    @Test
    public void execute_hasNoPath_pathNotFound() {
        relPathCommand = new RelPathCommand(INDEX_FIRST_PERSON, INDEX_SECOND_PERSON);
        relPathCommand.setData(model, null, null, null);
        String expectedMessage = String.format(MESSAGE_NO_PATH, ALICE.getName().fullName, BENSON.getName().fullName);

        assertCommandSuccess(relPathCommand, model, expectedMessage, expectedModel);
    }
}
```
###### /java/seedu/address/logic/parser/AddressBookParserTest.java
``` java
    @Test
    public void parseCommand_backup() throws Exception {
        assertTrue(parser.parseCommand(BackupCommand.COMMAND_WORD) instanceof BackupCommand);
        assertTrue(parser.parseCommand(BackupCommand.COMMAND_WORD + " 3") instanceof BackupCommand);
    }
```
###### /java/seedu/address/logic/parser/AddressBookParserTest.java
``` java
    @Test
    public void parseCommand_clearhistory() throws Exception {
        assertTrue(parser.parseCommand(ClearHistoryCommand.COMMAND_WORD) instanceof ClearHistoryCommand);
        assertTrue(parser.parseCommand(ClearHistoryCommand.COMMAND_WORD + " 3") instanceof ClearHistoryCommand);
    }

    @Test
    public void parseCommand_colourtag() throws Exception {
        final Tag tag = new Tag("test");
        final String colour = "red";
        ColourTagCommand command = (ColourTagCommand) parser.parseCommand(ColourTagCommand.COMMAND_WORD
                + " test red");
        assertEquals(new ColourTagCommand(tag, colour), command);
    }
```
###### /java/seedu/address/logic/parser/ColourTagCommandParserTest.java
``` java
public class ColourTagCommandParserTest {

    private ColourTagCommandParser parser = new ColourTagCommandParser();

    @Test
    public void parse_emptyArg_throwsParseException() {
        assertParseFailure(parser, "     ", String.format(MESSAGE_INVALID_COMMAND_FORMAT,
                ColourTagCommand.MESSAGE_USAGE));
    }

    @Test
    public void parse_invalidArgs_throwsParseException() {
        assertParseFailure(parser, "onlyonearg", String.format(MESSAGE_INVALID_COMMAND_FORMAT,
                ColourTagCommand.MESSAGE_USAGE));
        assertParseFailure(parser, "&&@&C@B invalidtag", String.format(MESSAGE_INVALID_COMMAND_FORMAT,
                ColourTagCommand.MESSAGE_USAGE));
    }

    @Test
    public void parse_validArgs_returnsColourTagCommand() throws Exception {
        assertParseSuccess(parser, "friends red", new ColourTagCommand(new Tag("friends"), "red"));
    }
}
```
###### /java/seedu/address/logic/parser/RelPathCommandParserTest.java
``` java
public class RelPathCommandParserTest {
    private RelPathCommandParser parser = new RelPathCommandParser();

    @Test
    public void parse_validArgs_returnsRelPathCommand() {
        assertParseSuccess(parser, "1 2", new RelPathCommand(INDEX_FIRST_PERSON, INDEX_SECOND_PERSON));
    }

    @Test
    public void parse_invalidArgs_throwsParseException() {
        assertParseFailure(parser, "1 a", String.format(MESSAGE_INVALID_COMMAND_FORMAT,
                RelPathCommand.MESSAGE_USAGE));
        assertParseFailure(parser, "a", String.format(MESSAGE_INVALID_COMMAND_FORMAT,
                RelPathCommand.MESSAGE_USAGE));
    }
}
```
###### /java/seedu/address/logic/parser/RemoveTagCommandParserTest.java
``` java
public class RemoveTagCommandParserTest {

    private RemoveTagCommandParser parser = new RemoveTagCommandParser();

    @Test
    public void parse_emptyArg_throwsParseException() {
        assertParseFailure(parser, "     ", String.format(MESSAGE_INVALID_COMMAND_FORMAT,
                RemoveTagCommand.MESSAGE_USAGE));
    }

    @Test
    public void parse_invalidArgs_throwsParseException() {
        assertParseFailure(parser, "!nvalid_T&g(", String.format(MESSAGE_INVALID_COMMAND_FORMAT,
                RemoveTagCommand.MESSAGE_USAGE));
    }

    @Test
    public void parse_validArgs_returnsRemoveTagCommand() {
        assertParseSuccess(parser, "validtag", new RemoveTagCommand("validtag"));
    }
}
```
###### /java/seedu/address/logic/UndoRedoStackTest.java
``` java
    @Test
    public void clear() {
        undoRedoStack = prepareStack(Collections.singletonList(dummyUndoableCommandOne),
                Arrays.asList(dummyUndoableCommandOne, dummyUndoableCommandTwo));
        undoRedoStack.clear();
        assertFalse(undoRedoStack.canUndo());
        assertFalse(undoRedoStack.canRedo());
    }
```
###### /java/seedu/address/relationship/ConfidenceEstimateTest.java
``` java
public class ConfidenceEstimateTest {

    @Test
    public void isValidConfidenceEstimate() {
        // non-number strings are not valid
        assertFalse(ConfidenceEstimate.isValidConfidenceEstimate("invalid"));

        // values below 0 and above 100 are not valid
        assertFalse(ConfidenceEstimate.isValidConfidenceEstimate("-1.0"));
        assertFalse(ConfidenceEstimate.isValidConfidenceEstimate("101.0"));

        // values between 0 and 100 inclusive are valid
        assertTrue(ConfidenceEstimate.isValidConfidenceEstimate("35.9"));
    }
}
```
###### /java/seedu/address/storage/JsonUserPrefsStorageTest.java
``` java
    private static Map<Tag, String> getSampleTagColours() {
        HashMap<Tag, String> sampleTagColours = new HashMap<>();
        try {
            sampleTagColours.put(new Tag("friends"), "red");
            sampleTagColours.put(new Tag("colleagues"), "green");
            sampleTagColours.put(new Tag("family"), "yellow");
            sampleTagColours.put(new Tag("neighbours"), "blue");
        } catch (IllegalValueException e) {
            throw new AssertionError("sample data cannot be invalid", e);
        }

        return sampleTagColours;
    }
```
###### /java/seedu/address/storage/StorageManagerTest.java
``` java
    @Test
    public void backupAddressBookReadSave() throws Exception {
        /*
         * Note: This is an integration test that verifies the StorageManager is properly wired to the
         * {@link XmlAddressBookStorage} class.
         * More extensive testing of UserPref saving/reading is done in {@link XmlAddressBookStorageTest} class.
         */
        AddressBook original = getTypicalAddressBookWithRelationships();
        storageManager.backupAddressBook(original);
        ReadOnlyAddressBook retrieved = storageManager.readBackupAddressBook().get();
        assertEquals(original, new AddressBook(retrieved));
    }
```
###### /java/seedu/address/storage/XmlAddressBookStorageTest.java
``` java
        //Backup address book and read back
        xmlAddressBookStorage.backupAddressBook(original);
        readBack = xmlAddressBookStorage.readBackupAddressBook().get();
        assertEquals(original, new AddressBook(readBack));
```
###### /java/seedu/address/testutil/StorageStub.java
``` java
/**
 * A default storage stub that has all of the methods failing.
 */
public class StorageStub implements Storage {
    @Override
    public Optional<UserPrefs> readUserPrefs() throws DataConversionException, IOException {
        fail("This method should not be called.");
        return null;
    }

    @Override
    public void saveUserPrefs(UserPrefs userPrefs) throws IOException {
        fail("This method should not be called.");
    }

    @Override
    public String getAddressBookFilePath() {
        fail("This method should not be called.");
        return null;
    }

    @Override
    public Optional<ReadOnlyAddressBook> readAddressBook() throws DataConversionException, IOException {
        fail("This method should not be called.");
        return null;
    }

    @Override
    public Optional<ReadOnlyAddressBook> readAddressBook(String filePath) throws DataConversionException, IOException {
        fail("This method should not be called.");
        return null;
    }

    @Override
    public void saveAddressBook(ReadOnlyAddressBook addressBook) throws IOException {
        fail("This method should not be called.");
    }

    @Override
    public void saveAddressBook(ReadOnlyAddressBook addressBook, String filePath) throws IOException {
        fail("This method should not be called.");
    }

    @Override
    public void handleAddressBookChangedEvent(AddressBookChangedEvent abce) {
        fail("This method should not be called.");
    }

    @Override
    public String getBackupAddressBookFilePath() {
        fail("This method should not be called.");
        return null;
    }

    @Override
    public Optional<ReadOnlyAddressBook> readBackupAddressBook() throws DataConversionException, IOException {
        fail("This method should not be called.");
        return null;
    }

    @Override
    public void backupAddressBook(ReadOnlyAddressBook addressBook) throws IOException {
        fail("This method should not be called.");
    }

    @Override
    public String getUserPrefsFilePath() {
        fail("This method should not be called.");
        return null;
    }
}
```
###### /java/seedu/address/testutil/TypicalPersons.java
``` java
    // These persons have relationships for the purpose of testing storage mechanisms.
    // they are separate so as not to affect other tests that assume persons are independent eg. system tests
    public static final ReadOnlyPerson JAMES = new PersonBuilder().withName("James King").withPhone("65637492")
            .withEmail("jamesk@example.com").withAddress("123 Griffin Rd").build();
    public static final ReadOnlyPerson KELVIN = new PersonBuilder().withName("Kelvin Klein").withPhone("98762456")
            .withEmail("kklein@example.com").withAddress("23 Mobius Strip").build();
    public static final ReadOnlyPerson LISA = new PersonBuilder().withName("Lisa McAvoy").withPhone("65243679")
            .withEmail("lisamc@example.com").withAddress("43 Trance Ave").build();

    public static final String KEYWORD_MATCHING_MEIER = "Meier"; // A keyword that matches MEIER

    /**
     * Static initializer is for adding relationships to persons after they are all instantiated.
     */
    static {
        try {
            Relationship undirectedFrom = new Relationship(JAMES, KELVIN, RelationshipDirection.UNDIRECTED,
                    new Name("undirected"), new ConfidenceEstimate(29));
            Relationship undirectedTo = new Relationship(KELVIN, JAMES, RelationshipDirection.UNDIRECTED,
                    new Name("undirected"), new ConfidenceEstimate(29));
            Relationship directed = new Relationship(KELVIN, LISA, RelationshipDirection.DIRECTED,
                    new Name("directed"), new ConfidenceEstimate(30));

            Person james = (Person) JAMES;
            Person kelvin = (Person) KELVIN;
            Person lisa = (Person) LISA;

            james.addRelationship(undirectedFrom);
            kelvin.addRelationship(undirectedTo);
            kelvin.addRelationship(directed);
            lisa.addRelationship(directed);
        } catch (DuplicateRelationshipException | IllegalValueException e) {
            assert false : "impossible to have invalid data in test data";
        }
    }
```
###### /java/seedu/address/testutil/TypicalPersons.java
``` java
    /**
     * Returns an {@code AddressBook} with all the typical persons sorted by address.
     */
    public static AddressBook getTypicalSortedAddressAddressBook() {
        AddressBook ab = new AddressBook();
        for (ReadOnlyPerson person : getTypicalSortedAddressPersons()) {
            try {
                ab.addPerson(person);
            } catch (DuplicatePersonException e) {
                assert false : "not possible";
            }
        }
        return ab;
    }

    /**
     * Returns an {@code AddressBook} with all the typical persons sorted by remark.
     */
    public static AddressBook getTypicalSortedRemarkAddressBook() {
        AddressBook ab = new AddressBook();
        for (ReadOnlyPerson person : getTypicalSortedRemarkPersons()) {
            try {
                ab.addPerson(person);
            } catch (DuplicatePersonException e) {
                assert false : "not possible";
            }
        }
        return ab;
    }

    /**
     * Returns an {@code AddressBook} with all the typical persons in unsorted order by name.
     */
    public static AddressBook getUnsortedNameAddressBook() {
        AddressBook ab = new AddressBook();
        for (ReadOnlyPerson person : getUnsortedNameTypicalPersons()) {
            try {
                ab.addPerson(person);
            } catch (DuplicatePersonException e) {
                assert false : "not possible";
            }
        }
        return ab;
    }

    /**
     * Returns an {@code AddressBook} with all the typical persons in unsorted order by phone.
     */
    public static AddressBook getUnsortedPhoneAddressBook() {
        AddressBook ab = new AddressBook();
        for (ReadOnlyPerson person : getUnsortedPhoneTypicalPersons()) {
            try {
                ab.addPerson(person);
            } catch (DuplicatePersonException e) {
                assert false : "not possible";
            }
        }
        return ab;
    }

    /**
     * Returns an {@code AddressBook} with all the typical persons in unsorted order by email.
     */
    public static AddressBook getUnsortedEmailAddressBook() {
        AddressBook ab = new AddressBook();
        for (ReadOnlyPerson person : getUnsortedEmailTypicalPersons()) {
            try {
                ab.addPerson(person);
            } catch (DuplicatePersonException e) {
                assert false : "not possible";
            }
        }
        return ab;
    }

    /**
     * Returns an {@code AddressBook} with all the typical persons in unsorted order by address.
     */
    public static AddressBook getUnsortedAddressAddressBook() {
        AddressBook ab = new AddressBook();
        for (ReadOnlyPerson person : getUnsortedAddressTypicalPersons()) {
            try {
                ab.addPerson(person);
            } catch (DuplicatePersonException e) {
                assert false : "not possible";
            }
        }
        return ab;
    }

    /**
     * Returns an {@code AddressBook} with all the typical persons in unsorted order by address.
     */
    public static AddressBook getUnsortedRemarkAddressBook() {
        AddressBook ab = new AddressBook();
        for (ReadOnlyPerson person : getUnsortedRemarkTypicalPersons()) {
            try {
                ab.addPerson(person);
            } catch (DuplicatePersonException e) {
                assert false : "not possible";
            }
        }
        return ab;
    }
```
###### /java/seedu/address/testutil/TypicalPersons.java
``` java
    public static List<ReadOnlyPerson> getTypicalPersonsWithRelationships() {
        return new ArrayList<>(Arrays.asList(ALICE, BENSON, CARL, DANIEL, ELLE, FIONA, GEORGE, JAMES, KELVIN, LISA));
    }
```
