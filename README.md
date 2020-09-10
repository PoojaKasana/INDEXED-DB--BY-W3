# INDEXED-DB--BY-W3

Indexed DB is a web storage API by which we can store huge data of web using browsers and JS platforms(NODE JS). Docs here-> https://developer.mozilla.org/en-US/docs/Web/API/IndexedDB_API

We can query data for specific platform using inbuilt methods. Notice the docs gives methods which are not promised based(we cant handle by .then/.catch or async await). So I used IDB promise and created a  helper model of indexed DB for our project . Here it goes->

import { openDB, deleteDB, DBSchema } from "idb";

interface MyDB extends DBSchema {
  currentTabValue: {
    key: string;
    value: string;
  };
}
interface IDBStorageService {
  myDB: any;
  storeName: string;
}

class DBStorageService implements IDBStorageService {
  public myDB: any;
  public storeName: string;
  public constructor(storeName) {
    this.myDB = {};
    this.storeName = storeName;
  }

  //chcek if the database exists
  public static async checkIfDBExists(dbName, callback) {
    let dbExists = true;

    let request = window.indexedDB.open(dbName);
    request.onsuccess = function() {
      request.result.close();
      if (!dbExists) indexedDB.deleteDatabase(dbName);
      callback(dbExists);
    };
    request.onupgradeneeded = function() {
      dbExists = false;
    };
  }

  //factory method
  public static createDBStorageService(store: string) {
    return new DBStorageService(store);
  }

  //Create a Database
  public async createDB(dbName, versionNum, store = this.storeName) {
    this.myDB = await openDB<any>(dbName, versionNum, {
      upgrade(db) {
        db.createObjectStore(store);
      }
    });
  }

  //Drop the database
  public async dropDB(dbName) {
    await deleteDB(dbName);
  }

  //Fetch key from the store in the DataBase
  public async get(mydb = this.myDB, mystore = this.storeName, key) {
    return mydb ? (await mydb).get(mystore, key) : null;
  }

  //Delete A Key in the store from the DataBase
  public async delete(mydb = this.myDB, mystore = this.storeName, key) {
    return mydb ? (await mydb).delete(mystore, key) : null;
  }

  //Clear the key in the store from the Database
  public async clear(mydb = this.myDB, mystore = this.storeName) {
    return mydb ? (await mydb).clear(mystore) : null;
  }

  //get all keys in the store
  public async keys(mydb = this.myDB, mystore = this.storeName) {
    return mydb ? (await mydb).getAllKeys(mystore) : null;
  }

  //set value to a key in the store
  public async set(mydb = this.myDB, mystore = this.storeName, key, val) {
    return mydb ? (await mydb).put(mystore, val, key) : null;
  }
}

export default DBStorageService;


Now it is promised based. we can do .then/catch easily . Usage here->

//Clear local storage
  useEffect(() => {
    // eslint-disable-next-line no-console
    console.log("mounted here");

    DBStorageService.checkIfDBExists(TAKE_NOTES_DB, function(exists) {
      // eslint-disable-next-line no-console
      console.log(exists);
      if (exists) return;
      DBStorageService.createDBStorageService(TAKE_NOTES_DB);
    });

    return () => {
      // eslint-disable-next-line no-console
      console.log("un mounted here");
      DBStorageService.createDBStorageService(TAKE_NOTES_DB);
    };
  }, []);
  
  
