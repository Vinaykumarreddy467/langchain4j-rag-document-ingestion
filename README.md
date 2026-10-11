# LangChain4j RAG Document Ingestion

A Java ingestion pipeline for RAG (Retrieval-Augmented Generation). It extracts text from eight
document formats using [LangChain4j](https://github.com/langchain4j/langchain4j) document parsers,
then splits the text into overlapping, token-sized chunks ready for embedding.

Use it as a command-line tool or as a library inside your own Java application.

---

## Features

- **Multi-format extraction.** PDF, Word (`.docx`, `.doc`), PowerPoint, Excel, plain text, HTML and Markdown.
- **Batch mode.** Point it at a directory and it extracts every supported file, skips unsupported ones, and prints a summary.
- **RAG-ready chunking.** Uses LangChain4j's recursive splitter: 512 tokens per chunk with a 50-token overlap by default, and both are configurable.
- **Extraction metadata.** Each result carries the text, word count, file metadata and success or failure status.
- **Saved output.** In single-file mode, the full extracted text is written to `output/<file-name>_extracted.txt`.
- **Tested.** A JUnit 5 suite covers extraction, chunking, directory scanning and error handling.

---

## Supported formats

| Format     | Extension       | Parser                       |
|------------|-----------------|------------------------------|
| PDF        | `.pdf`          | `ApachePdfBoxDocumentParser` |
| Word       | `.docx`         | `ApachePoiDocumentParser`    |
| Word       | `.doc`          | `ApachePoiDocumentParser`    |
| PowerPoint | `.pptx`         | `ApachePoiDocumentParser`    |
| Excel      | `.xlsx`         | `ApachePoiDocumentParser`    |
| Plain text | `.txt`          | `TextDocumentParser`         |
| HTML       | `.html`, `.htm` | `TextDocumentParser`         |
| Markdown   | `.md`           | `TextDocumentParser`         |

---

## Project structure

```
src/main/java/com/vinay/ragingestion/
├── DocumentIngestionApp.java          ← CLI entry point
├── model/
│   ├── DocumentType.java              ← Enum: PDF, DOCX, PPTX, etc.
│   └── ExtractionResult.java          ← Result with text + metadata
├── parser/
│   ├── DocumentParserFactory.java     ← Picks the right LangChain4j parser
│   └── UnsupportedDocumentTypeException.java
├── service/
│   ├── DocumentExtractionService.java ← Core extraction logic
│   └── TextChunkingService.java       ← RAG-ready text chunking
└── util/
    └── FileExtensionUtil.java         ← File helper utilities
```

---

## Build and run

### Prerequisites

- Java 17+
- Maven 3.8+

### Build the fat JAR

```bash
mvn clean package -DskipTests
```

This produces `target/langchain4j-rag-document-ingestion-1.0.0.jar`.

### Run

**Single file:**

```bash
java -jar target/langchain4j-rag-document-ingestion-1.0.0.jar /path/to/file.pdf
```

**Directory (batch):**

```bash
java -jar target/langchain4j-rag-document-ingestion-1.0.0.jar /path/to/docs/
```

**Extract and chunk for RAG:**

```bash
java -jar target/langchain4j-rag-document-ingestion-1.0.0.jar --chunk /path/to/file.pdf
```

### Run the tests

```bash
mvn test
```

---

## Use as a library

```java
// Extract a single file
DocumentExtractionService service = new DocumentExtractionService();
ExtractionResult result = service.extract("/path/to/resume.pdf");

if (result.isSuccess()) {
    String text     = result.getExtractedText();
    int wordCount   = result.getWordCount();
    Map<String,String> meta = result.getMetadata();
}

// Batch extract a directory
List<ExtractionResult> results = service.extractFromDirectory("/docs/");
service.printSummary(results);

// Chunk for RAG (default: 512 tokens, 50 overlap)
TextChunkingService chunker = new TextChunkingService();
List<String> chunks = chunker.chunkToStrings(result.getExtractedText());

// Or choose your own chunk size and overlap
TextChunkingService customChunker = new TextChunkingService(256, 32);
```

---

## Extending to local LLMs (Ollama)

You can pass extracted text straight to a local Ollama model:

```java
// Add to pom.xml: langchain4j-ollama
OllamaChatModel model = OllamaChatModel.builder()
        .baseUrl("http://localhost:11434")
        .modelName("llama3.1:8b")
        .build();

String answer = model.generate("Summarise this document:\n\n" + result.getExtractedText());
```

---

## LangChain4j version

`0.36.2`, a stable release with full Apache PDFBox and POI parser support.
