# Documented

**Documentation**: https://androiddevnotesdocumented.github.io/android-room-with-a-view-kotlin-documented/documentation/html/navigation.html

## Flow of Data for Automatic UI Updates (Reactive UI)

The automatic update is possible because we are using LiveData. In the `MainActivity`, there is an `Observer` that observes the words LiveData from the database and is notified when they change. 

```kotlin
// MainActivity.kt

wordViewModel.allWords.observe(owner = this) { words ->
    // Update the cached copy of the words in the adapter.
    words.let { adapter.submitList(it) }
}

```

When there is a change, the observer's `onChange()` (the default method for our Lambda) method is executed and updates `words` in the `WordListAdapter`.

The data can be observed because it is `LiveData`. And what is observed is the `LiveData<List<Word>>` that is returned by the `WordViewModel` `allWords` property.


```kotlin
// WordViewModel.kt

val allWords: LiveData<List<Word>> = repository.allWords.asLiveData()

```

The `WordViewModel` hides everything about the backend from the UI layer. It provides methods for accessing the data layer, and it returns `LiveData` so that `MainActivity` can set up the observer relationship.

`Views` and `Activities` (and `Fragments`) only interact with the data through the `ViewModel`. As such, it doesn't matter where the data comes from.

In this case, the data comes from a `Repository`. The `ViewModel` does not need to know what that Repository interacts with. It just needs to know how to interact with the `Repository`, which is through the [methods exposed](https://androiddevnotesdocumented.github.io/android-room-with-a-view-kotlin-documented/documentation/html/app/com.example.android.roomwordssample/-word-repository/index.html) by the `Repository`.

```kotlin
// WordViewModel.kt

fun insert(word: Word) = viewModelScope.launch {
    repository.insert(word)
}
```

The Repository manages one or more data sources. In the `WordListSample` app, that backend is a Room database. Room is a wrapper around and implements a SQLite database. Room does a lot of work for you that you used to have to do yourself. For example, Room does everything that you used to do with an `SQLiteOpenHelper` class.

```kotlin
// WordRepository.kt

class WordRepository(private val wordDao: WordDao) {

    val allWords: Flow<List<Word>> = wordDao.getAlphabetizedWords()

    suspend fun insert(word: Word) {
        wordDao.insert(word)
    }
}

```

The DAO maps method calls to database queries, so that when the Repository calls a method such as `getAlphabetizedWords()`, Room can execute `SELECT * FROM word_table ORDER BY word ASC`

```kotlin
// WordDao.kt

@Query("SELECT * FROM word_table ORDER BY word ASC")
fun getAlphabetizedWords(): Flow<List<Word>>

```

Because the result returned from the query is observed `Flow`, every time the data in Room changes, the `Observer` interface's `onChanged()` method is executed and the UI updated.