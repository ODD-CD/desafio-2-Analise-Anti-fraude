# desafio-2-Analise-Anti-fraude

arquivo de configurações para as credenciais do Azure:
```
# src/utils/Config.py 
from dotenv import load_dotenv 
import os 
load_dotenv() 
class Config: 
    # Azure Document Intelligence 
    ENDPOINT =  os.getenv("AZURE_DOC_INT_ENDPOINT")
    KEY = os.getenv("AZURE_DOC_INT_KEY")
    # Azure Blob Storoge
    STORAGE_CONNECTION = os.getenv("AZURE_STORAGE_CONNECTION")
    CONTAINER_NAME = os.getenv("CONTAINER_NAME")
    # SQLite
    DATABASE_PATH = "desafio_2/data/cards.db"
```

codigo do banco de dados SQLite:

```
# src/services/data_base.py
import sqlite3
from typing import Dict, List, Optional
from src.utils.Config import Config 

class DataBase:
    def __init__(self):
        self.db_path = Config.DATABASE_PATH
        # self._create_table()
    
    def __init__(self):
        self.db_path = Config.DATABASE_PATH
        
        with sqlite3.connect(self.db_path) as conn:
            # Criar a tabela se não existir
            cursor = conn.cursor()
            cursor.execute('''
                CREATE TABLE IF NOT EXISTS credit_cards (
                    id INTEGER PRIMARY KEY AUTOINCREMENT,
                    card_name TEXT,
                    card_number TEXT,
                    expiry_date TEXT,
                    bank_name TEXT,
                    is_valid TEXT,
                    processed_at TEXT
                )
            ''')
            conn.commit()
    
    def _execute_query(self, query: str, params: tuple = None) -> sqlite3.Cursor:
        """Executa uma query SQL."""
        with sqlite3.connect(self.db_path) as conn:
            cursor = conn.cursor()
            if params:
                cursor.execute(query, params)
            else:
                cursor.execute(query)
            conn.commit()
            return cursor
    
    def insert_card(self, card_info: Dict[str, str]) -> int:
        """Insere um novo cartão no banco."""
        query = '''
        INSERT INTO credit_cards (
            card_name, card_number, expiry_date, 
            bank_name, is_valid, processed_at
        ) VALUES (?, ?, ?, ?, ?, ?)
        '''
        cursor = self._execute_query(query, (
            card_info['card_name'],
            card_info['card_number'],
            card_info['expiry_date'],
            card_info['bank_name'],
            card_info['is_valid'],
            card_info['processed_at']
        ))
        return cursor.lastrowid
    
    def get_all_cards(self) -> List[Dict[str, str]]:
        """Retorna todos os cartões."""
        query = "SELECT * FROM credit_cards"
        cursor = self._execute_query(query)
        columns = [desc[0] for desc in cursor.description]
        return [dict(zip(columns, row)) for row in cursor.fetchall()]
    
    def get_card_by_id(self, card_id: int) -> Optional[Dict[str, str]]:
        """Retorna um cartão específico."""
        query = "SELECT * FROM credit_cards WHERE id = ?"
        cursor = self._execute_query(query, (card_id,))
        row = cursor.fetchone()
        if row:
            columns = [desc[0] for desc in cursor.description]
            return dict(zip(columns, row))
        return None
    
    def update_card(self, card_id: int, card_info: Dict[str, str]) -> bool:
        """Atualiza um cartão existente."""
        query = '''
        UPDATE credit_cards
        SET card_name = ?, card_number = ?, expiry_date = ?,
            bank_name = ?, is_valid = ?, processed_at = ?
        WHERE id = ?
        '''
        cursor = self._execute_query(query, (
            card_info['card_name'],
            card_info['card_number'],
            card_info['expiry_date'],
            card_info['bank_name'],
            card_info['is_valid'],
            card_info['processed_at'],
            card_id
        ))
        return cursor.rowcount > 0
    
    def delete_card(self, card_id: int) -> bool:
        """Deleta um cartão."""
        query = "DELETE FROM credit_cards WHERE id = ?"
        cursor = self._execute_query(query, (card_id,))
        return cursor.rowcount > 0
```
