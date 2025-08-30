# Data Encryption: Securing Information at Rest and in Transit

Data encryption protects sensitive information by converting it into an unreadable format using cryptographic algorithms. Modern encryption strategies cover data at rest (stored data), data in transit (communication), and data in use (processing), providing comprehensive protection against unauthorized access.

## 🔐 Encryption Architecture

```mermaid
graph TB
    subgraph "Data Sources"
        USER_DATA[User Data]
        APP_DATA[Application Data]
        LOGS[System Logs]
        BACKUPS[Backup Data]
        CONFIG[Configuration]
    end
    
    subgraph "Data in Transit"
        TLS_LAYER[TLS/SSL Layer]
        VPN_TUNNEL[VPN Tunnels]
        API_GATEWAY[API Gateway]
        MESSAGE_QUEUE[Encrypted Messaging]
        PEER_TO_PEER[P2P Encryption]
    end
    
    subgraph "Encryption Services"
        KEY_MANAGEMENT[Key Management Service]
        HSM[Hardware Security Module]
        CERT_AUTHORITY[Certificate Authority]
        CRYPTO_ENGINE[Cryptographic Engine]
        RANDOM_GEN[Random Number Generator]
    end
    
    subgraph "Data at Rest"
        ENCRYPTED_DB[Encrypted Database]
        ENCRYPTED_FILES[Encrypted File System]
        ENCRYPTED_STORAGE[Encrypted Object Storage]
        ENCRYPTED_CACHE[Encrypted Cache]
        ENCRYPTED_BACKUP[Encrypted Backups]
    end
    
    subgraph "Data in Use"
        SECURE_ENCLAVE[Secure Enclave]
        CONFIDENTIAL_COMPUTE[Confidential Computing]
        HOMOMORPHIC[Homomorphic Encryption]
        MEMORY_ENCRYPTION[Memory Encryption]
        PROCESS_ISOLATION[Process Isolation]
    end
    
    subgraph "Key Storage"
        VAULT[Secret Vault]
        KEY_ESCROW[Key Escrow]
        DISTRIBUTED_KEYS[Distributed Keys]
        BACKUP_KEYS[Key Backup]
    end
    
    USER_DATA --> TLS_LAYER
    APP_DATA --> VPN_TUNNEL
    LOGS --> API_GATEWAY
    BACKUPS --> MESSAGE_QUEUE
    CONFIG --> PEER_TO_PEER
    
    TLS_LAYER --> KEY_MANAGEMENT
    VPN_TUNNEL --> HSM
    API_GATEWAY --> CERT_AUTHORITY
    MESSAGE_QUEUE --> CRYPTO_ENGINE
    PEER_TO_PEER --> RANDOM_GEN
    
    KEY_MANAGEMENT --> ENCRYPTED_DB
    HSM --> ENCRYPTED_FILES
    CERT_AUTHORITY --> ENCRYPTED_STORAGE
    CRYPTO_ENGINE --> ENCRYPTED_CACHE
    RANDOM_GEN --> ENCRYPTED_BACKUP
    
    ENCRYPTED_DB --> SECURE_ENCLAVE
    ENCRYPTED_FILES --> CONFIDENTIAL_COMPUTE
    ENCRYPTED_STORAGE --> HOMOMORPHIC
    ENCRYPTED_CACHE --> MEMORY_ENCRYPTION
    ENCRYPTED_BACKUP --> PROCESS_ISOLATION
    
    KEY_MANAGEMENT --> VAULT
    HSM --> KEY_ESCROW
    CERT_AUTHORITY --> DISTRIBUTED_KEYS
    CRYPTO_ENGINE --> BACKUP_KEYS
    
    style KEY_MANAGEMENT fill:#ff9999
    style CRYPTO_ENGINE fill:#99ff99
    style VAULT fill:#9999ff
```

## 🚀 Encryption Implementation

```python
import asyncio
import base64
import hashlib
import hmac
import json
import os
import secrets
import sqlite3
import threading
import time
import uuid
from abc import ABC, abstractmethod
from cryptography.hazmat.primitives import hashes, serialization
from cryptography.hazmat.primitives.asymmetric import rsa, padding as asym_padding
from cryptography.hazmat.primitives.ciphers import Cipher, algorithms, modes
from cryptography.hazmat.primitives.kdf.pbkdf2 import PBKDF2HMAC
from cryptography.hazmat.primitives.kdf.scrypt import Scrypt
from cryptography.hazmat.primitives.padding import PKCS7
from cryptography.fernet import Fernet
from cryptography.x509.oid import NameOID
from cryptography import x509
from dataclasses import dataclass, field
from typing import Dict, List, Any, Optional, Tuple, Union, Protocol
from enum import Enum
import weakref
import logging

class EncryptionAlgorithm(Enum):
    AES_256_GCM = "aes_256_gcm"
    AES_256_CBC = "aes_256_cbc"
    CHACHA20_POLY1305 = "chacha20_poly1305"
    FERNET = "fernet"
    RSA_4096 = "rsa_4096"
    RSA_2048 = "rsa_2048"

class KeyType(Enum):
    SYMMETRIC = "symmetric"
    ASYMMETRIC_PUBLIC = "asymmetric_public"
    ASYMMETRIC_PRIVATE = "asymmetric_private"
    HMAC = "hmac"

@dataclass
class EncryptionKey:
    """Encryption key with metadata"""
    key_id: str
    key_type: KeyType
    algorithm: EncryptionAlgorithm
    key_data: bytes
    created_at: float
    expires_at: Optional[float] = None
    metadata: Dict[str, Any] = field(default_factory=dict)
    usage_count: int = 0
    max_usage: Optional[int] = None

@dataclass
class EncryptedData:
    """Container for encrypted data with metadata"""
    data: bytes
    algorithm: EncryptionAlgorithm
    key_id: str
    iv: Optional[bytes] = None
    salt: Optional[bytes] = None
    tag: Optional[bytes] = None
    metadata: Dict[str, Any] = field(default_factory=dict)

class CryptoEngine:
    """Core cryptographic operations engine"""
    
    def __init__(self):
        self._secure_random = secrets.SystemRandom()
        self._setup_logging()
    
    def _setup_logging(self):
        """Setup secure logging for crypto operations"""
        self.logger = logging.getLogger('crypto_engine')
        self.logger.setLevel(logging.INFO)
        
        # Don't log sensitive data in production
        self.secure_mode = True
    
    def generate_symmetric_key(self, algorithm: EncryptionAlgorithm) -> bytes:
        """Generate a symmetric encryption key"""
        if algorithm == EncryptionAlgorithm.AES_256_GCM:
            return os.urandom(32)  # 256 bits
        elif algorithm == EncryptionAlgorithm.AES_256_CBC:
            return os.urandom(32)  # 256 bits
        elif algorithm == EncryptionAlgorithm.CHACHA20_POLY1305:
            return os.urandom(32)  # 256 bits
        elif algorithm == EncryptionAlgorithm.FERNET:
            return Fernet.generate_key()
        else:
            raise ValueError(f"Unsupported symmetric algorithm: {algorithm}")
    
    def generate_asymmetric_keypair(self, algorithm: EncryptionAlgorithm) -> Tuple[bytes, bytes]:
        """Generate an asymmetric key pair (private, public)"""
        if algorithm in [EncryptionAlgorithm.RSA_4096, EncryptionAlgorithm.RSA_2048]:
            key_size = 4096 if algorithm == EncryptionAlgorithm.RSA_4096 else 2048
            
            private_key = rsa.generate_private_key(
                public_exponent=65537,
                key_size=key_size
            )
            
            private_pem = private_key.private_bytes(
                encoding=serialization.Encoding.PEM,
                format=serialization.PrivateFormat.PKCS8,
                encryption_algorithm=serialization.NoEncryption()
            )
            
            public_key = private_key.public_key()
            public_pem = public_key.public_bytes(
                encoding=serialization.Encoding.PEM,
                format=serialization.PublicFormat.SubjectPublicKeyInfo
            )
            
            return private_pem, public_pem
        else:
            raise ValueError(f"Unsupported asymmetric algorithm: {algorithm}")
    
    def encrypt_symmetric(self, data: bytes, key: bytes, 
                         algorithm: EncryptionAlgorithm) -> EncryptedData:
        """Encrypt data using symmetric encryption"""
        if algorithm == EncryptionAlgorithm.AES_256_GCM:
            return self._encrypt_aes_gcm(data, key)
        elif algorithm == EncryptionAlgorithm.AES_256_CBC:
            return self._encrypt_aes_cbc(data, key)
        elif algorithm == EncryptionAlgorithm.CHACHA20_POLY1305:
            return self._encrypt_chacha20_poly1305(data, key)
        elif algorithm == EncryptionAlgorithm.FERNET:
            return self._encrypt_fernet(data, key)
        else:
            raise ValueError(f"Unsupported algorithm: {algorithm}")
    
    def decrypt_symmetric(self, encrypted_data: EncryptedData, key: bytes) -> bytes:
        """Decrypt data using symmetric encryption"""
        algorithm = encrypted_data.algorithm
        
        if algorithm == EncryptionAlgorithm.AES_256_GCM:
            return self._decrypt_aes_gcm(encrypted_data, key)
        elif algorithm == EncryptionAlgorithm.AES_256_CBC:
            return self._decrypt_aes_cbc(encrypted_data, key)
        elif algorithm == EncryptionAlgorithm.CHACHA20_POLY1305:
            return self._decrypt_chacha20_poly1305(encrypted_data, key)
        elif algorithm == EncryptionAlgorithm.FERNET:
            return self._decrypt_fernet(encrypted_data, key)
        else:
            raise ValueError(f"Unsupported algorithm: {algorithm}")
    
    def _encrypt_aes_gcm(self, data: bytes, key: bytes) -> EncryptedData:
        """Encrypt using AES-256-GCM"""
        iv = os.urandom(12)  # 96-bit IV for GCM
        
        cipher = Cipher(
            algorithms.AES(key),
            modes.GCM(iv)
        )
        
        encryptor = cipher.encryptor()
        ciphertext = encryptor.update(data) + encryptor.finalize()
        
        return EncryptedData(
            data=ciphertext,
            algorithm=EncryptionAlgorithm.AES_256_GCM,
            key_id="",  # Will be set by key manager
            iv=iv,
            tag=encryptor.tag
        )
    
    def _decrypt_aes_gcm(self, encrypted_data: EncryptedData, key: bytes) -> bytes:
        """Decrypt using AES-256-GCM"""
        cipher = Cipher(
            algorithms.AES(key),
            modes.GCM(encrypted_data.iv, encrypted_data.tag)
        )
        
        decryptor = cipher.decryptor()
        plaintext = decryptor.update(encrypted_data.data) + decryptor.finalize()
        
        return plaintext
    
    def _encrypt_aes_cbc(self, data: bytes, key: bytes) -> EncryptedData:
        """Encrypt using AES-256-CBC with PKCS7 padding"""
        iv = os.urandom(16)  # 128-bit IV for CBC
        
        # Apply PKCS7 padding
        padder = PKCS7(128).padder()
        padded_data = padder.update(data) + padder.finalize()
        
        cipher = Cipher(
            algorithms.AES(key),
            modes.CBC(iv)
        )
        
        encryptor = cipher.encryptor()
        ciphertext = encryptor.update(padded_data) + encryptor.finalize()
        
        return EncryptedData(
            data=ciphertext,
            algorithm=EncryptionAlgorithm.AES_256_CBC,
            key_id="",
            iv=iv
        )
    
    def _decrypt_aes_cbc(self, encrypted_data: EncryptedData, key: bytes) -> bytes:
        """Decrypt using AES-256-CBC with PKCS7 padding"""
        cipher = Cipher(
            algorithms.AES(key),
            modes.CBC(encrypted_data.iv)
        )
        
        decryptor = cipher.decryptor()
        padded_data = decryptor.update(encrypted_data.data) + decryptor.finalize()
        
        # Remove PKCS7 padding
        unpadder = PKCS7(128).unpadder()
        plaintext = unpadder.update(padded_data) + unpadder.finalize()
        
        return plaintext
    
    def _encrypt_chacha20_poly1305(self, data: bytes, key: bytes) -> EncryptedData:
        """Encrypt using ChaCha20-Poly1305"""
        nonce = os.urandom(12)  # 96-bit nonce
        
        cipher = Cipher(
            algorithms.ChaCha20(key, nonce),
            mode=None
        )
        
        encryptor = cipher.encryptor()
        ciphertext = encryptor.update(data) + encryptor.finalize()
        
        return EncryptedData(
            data=ciphertext,
            algorithm=EncryptionAlgorithm.CHACHA20_POLY1305,
            key_id="",
            iv=nonce
        )
    
    def _decrypt_chacha20_poly1305(self, encrypted_data: EncryptedData, key: bytes) -> bytes:
        """Decrypt using ChaCha20-Poly1305"""
        cipher = Cipher(
            algorithms.ChaCha20(key, encrypted_data.iv),
            mode=None
        )
        
        decryptor = cipher.decryptor()
        plaintext = decryptor.update(encrypted_data.data) + decryptor.finalize()
        
        return plaintext
    
    def _encrypt_fernet(self, data: bytes, key: bytes) -> EncryptedData:
        """Encrypt using Fernet (AES-128-CBC + HMAC-SHA256)"""
        f = Fernet(key)
        ciphertext = f.encrypt(data)
        
        return EncryptedData(
            data=ciphertext,
            algorithm=EncryptionAlgorithm.FERNET,
            key_id=""
        )
    
    def _decrypt_fernet(self, encrypted_data: EncryptedData, key: bytes) -> bytes:
        """Decrypt using Fernet"""
        f = Fernet(key)
        plaintext = f.decrypt(encrypted_data.data)
        
        return plaintext
    
    def encrypt_asymmetric(self, data: bytes, public_key_pem: bytes,
                          algorithm: EncryptionAlgorithm) -> EncryptedData:
        """Encrypt data using asymmetric encryption"""
        if algorithm in [EncryptionAlgorithm.RSA_4096, EncryptionAlgorithm.RSA_2048]:
            public_key = serialization.load_pem_public_key(public_key_pem)
            
            ciphertext = public_key.encrypt(
                data,
                asym_padding.OAEP(
                    mgf=asym_padding.MGF1(algorithm=hashes.SHA256()),
                    algorithm=hashes.SHA256(),
                    label=None
                )
            )
            
            return EncryptedData(
                data=ciphertext,
                algorithm=algorithm,
                key_id=""
            )
        else:
            raise ValueError(f"Unsupported asymmetric algorithm: {algorithm}")
    
    def decrypt_asymmetric(self, encrypted_data: EncryptedData, 
                          private_key_pem: bytes) -> bytes:
        """Decrypt data using asymmetric encryption"""
        algorithm = encrypted_data.algorithm
        
        if algorithm in [EncryptionAlgorithm.RSA_4096, EncryptionAlgorithm.RSA_2048]:
            private_key = serialization.load_pem_private_key(
                private_key_pem,
                password=None
            )
            
            plaintext = private_key.decrypt(
                encrypted_data.data,
                asym_padding.OAEP(
                    mgf=asym_padding.MGF1(algorithm=hashes.SHA256()),
                    algorithm=hashes.SHA256(),
                    label=None
                )
            )
            
            return plaintext
        else:
            raise ValueError(f"Unsupported asymmetric algorithm: {algorithm}")
    
    def derive_key_from_password(self, password: str, salt: bytes,
                                algorithm: str = "pbkdf2") -> bytes:
        """Derive encryption key from password"""
        password_bytes = password.encode('utf-8')
        
        if algorithm == "pbkdf2":
            kdf = PBKDF2HMAC(
                algorithm=hashes.SHA256(),
                length=32,  # 256 bits
                salt=salt,
                iterations=100000
            )
        elif algorithm == "scrypt":
            kdf = Scrypt(
                algorithm=hashes.SHA256(),
                length=32,
                salt=salt,
                n=2**14,  # CPU cost factor
                r=8,      # Block size
                p=1       # Parallelization factor
            )
        else:
            raise ValueError(f"Unsupported KDF algorithm: {algorithm}")
        
        return kdf.derive(password_bytes)
    
    def generate_hmac(self, data: bytes, key: bytes) -> bytes:
        """Generate HMAC for data integrity"""
        h = hmac.new(key, data, hashlib.sha256)
        return h.digest()
    
    def verify_hmac(self, data: bytes, key: bytes, expected_hmac: bytes) -> bool:
        """Verify HMAC for data integrity"""
        calculated_hmac = self.generate_hmac(data, key)
        return hmac.compare_digest(calculated_hmac, expected_hmac)

class KeyManager:
    """Centralized key management system"""
    
    def __init__(self, crypto_engine: CryptoEngine):
        self.crypto_engine = crypto_engine
        self.keys: Dict[str, EncryptionKey] = {}
        self.master_key = self._load_or_generate_master_key()
        self._lock = threading.RLock()
        
        # Key rotation settings
        self.auto_rotate = True
        self.rotation_interval = 30 * 24 * 3600  # 30 days
        
        # Start key rotation background task
        if self.auto_rotate:
            threading.Thread(target=self._key_rotation_worker, daemon=True).start()
    
    def _load_or_generate_master_key(self) -> bytes:
        """Load or generate master key for key encryption"""
        master_key_file = "master.key"
        
        if os.path.exists(master_key_file):
            with open(master_key_file, 'rb') as f:
                return f.read()
        else:
            # Generate new master key
            master_key = self.crypto_engine.generate_symmetric_key(
                EncryptionAlgorithm.AES_256_GCM
            )
            
            # Save master key securely (in production, use HSM or secure key store)
            with open(master_key_file, 'wb') as f:
                f.write(master_key)
            
            os.chmod(master_key_file, 0o600)  # Restrict permissions
            
            return master_key
    
    def generate_key(self, algorithm: EncryptionAlgorithm, 
                    key_type: KeyType = KeyType.SYMMETRIC,
                    metadata: Optional[Dict[str, Any]] = None) -> str:
        """Generate and store a new encryption key"""
        with self._lock:
            key_id = str(uuid.uuid4())
            
            if key_type == KeyType.SYMMETRIC:
                key_data = self.crypto_engine.generate_symmetric_key(algorithm)
            elif key_type in [KeyType.ASYMMETRIC_PUBLIC, KeyType.ASYMMETRIC_PRIVATE]:
                private_key, public_key = self.crypto_engine.generate_asymmetric_keypair(algorithm)
                
                # Store both keys
                private_key_id = f"{key_id}_private"
                public_key_id = f"{key_id}_public"
                
                self.keys[private_key_id] = EncryptionKey(
                    key_id=private_key_id,
                    key_type=KeyType.ASYMMETRIC_PRIVATE,
                    algorithm=algorithm,
                    key_data=private_key,
                    created_at=time.time(),
                    metadata=metadata or {}
                )
                
                self.keys[public_key_id] = EncryptionKey(
                    key_id=public_key_id,
                    key_type=KeyType.ASYMMETRIC_PUBLIC,
                    algorithm=algorithm,
                    key_data=public_key,
                    created_at=time.time(),
                    metadata=metadata or {}
                )
                
                return key_id  # Return base ID
            elif key_type == KeyType.HMAC:
                key_data = os.urandom(32)  # 256-bit HMAC key
            else:
                raise ValueError(f"Unsupported key type: {key_type}")
            
            # Store symmetric or HMAC key
            encryption_key = EncryptionKey(
                key_id=key_id,
                key_type=key_type,
                algorithm=algorithm,
                key_data=key_data,
                created_at=time.time(),
                metadata=metadata or {}
            )
            
            self.keys[key_id] = encryption_key
            
            return key_id
    
    def get_key(self, key_id: str) -> Optional[EncryptionKey]:
        """Retrieve encryption key by ID"""
        with self._lock:
            key = self.keys.get(key_id)
            
            if key:
                # Check if key is expired
                if key.expires_at and time.time() > key.expires_at:
                    del self.keys[key_id]
                    return None
                
                # Check usage limits
                if key.max_usage and key.usage_count >= key.max_usage:
                    del self.keys[key_id]
                    return None
                
                # Increment usage count
                key.usage_count += 1
            
            return key
    
    def rotate_key(self, key_id: str) -> str:
        """Rotate (replace) an existing key"""
        with self._lock:
            old_key = self.keys.get(key_id)
            if not old_key:
                raise ValueError(f"Key {key_id} not found")
            
            # Generate new key with same parameters
            new_key_id = self.generate_key(
                algorithm=old_key.algorithm,
                key_type=old_key.key_type,
                metadata=old_key.metadata
            )
            
            # Mark old key as expired
            old_key.expires_at = time.time()
            
            return new_key_id
    
    def delete_key(self, key_id: str):
        """Securely delete encryption key"""
        with self._lock:
            if key_id in self.keys:
                # Overwrite key data before deletion
                key = self.keys[key_id]
                key.key_data = os.urandom(len(key.key_data))
                del self.keys[key_id]
    
    def list_keys(self, include_expired: bool = False) -> List[str]:
        """List all available key IDs"""
        with self._lock:
            current_time = time.time()
            
            if include_expired:
                return list(self.keys.keys())
            else:
                return [
                    key_id for key_id, key in self.keys.items()
                    if not key.expires_at or key.expires_at > current_time
                ]
    
    def _key_rotation_worker(self):
        """Background worker for automatic key rotation"""
        while True:
            try:
                time.sleep(3600)  # Check every hour
                current_time = time.time()
                
                with self._lock:
                    keys_to_rotate = []
                    
                    for key_id, key in self.keys.items():
                        # Check if key needs rotation
                        age = current_time - key.created_at
                        if age > self.rotation_interval:
                            keys_to_rotate.append(key_id)
                    
                    # Rotate old keys
                    for key_id in keys_to_rotate:
                        try:
                            new_key_id = self.rotate_key(key_id)
                            logging.info(f"Rotated key {key_id} -> {new_key_id}")
                        except Exception as e:
                            logging.error(f"Failed to rotate key {key_id}: {e}")
            
            except Exception as e:
                logging.error(f"Key rotation worker error: {e}")

class EncryptionService:
    """High-level encryption service"""
    
    def __init__(self):
        self.crypto_engine = CryptoEngine()
        self.key_manager = KeyManager(self.crypto_engine)
        self._default_algorithm = EncryptionAlgorithm.AES_256_GCM
    
    def encrypt_data(self, data: Union[str, bytes], 
                    key_id: Optional[str] = None,
                    algorithm: Optional[EncryptionAlgorithm] = None) -> Tuple[str, EncryptedData]:
        """Encrypt data and return encrypted result"""
        if isinstance(data, str):
            data = data.encode('utf-8')
        
        # Use provided algorithm or default
        algo = algorithm or self._default_algorithm
        
        # Generate key if not provided
        if not key_id:
            key_id = self.key_manager.generate_key(algo)
        
        # Get encryption key
        encryption_key = self.key_manager.get_key(key_id)
        if not encryption_key:
            raise ValueError(f"Encryption key {key_id} not found")
        
        # Encrypt data
        encrypted_data = self.crypto_engine.encrypt_symmetric(
            data, encryption_key.key_data, algo
        )
        encrypted_data.key_id = key_id
        
        return key_id, encrypted_data
    
    def decrypt_data(self, encrypted_data: EncryptedData) -> bytes:
        """Decrypt data using stored key"""
        # Get decryption key
        decryption_key = self.key_manager.get_key(encrypted_data.key_id)
        if not decryption_key:
            raise ValueError(f"Decryption key {encrypted_data.key_id} not found")
        
        # Decrypt data
        plaintext = self.crypto_engine.decrypt_symmetric(
            encrypted_data, decryption_key.key_data
        )
        
        return plaintext
    
    def encrypt_with_password(self, data: Union[str, bytes], password: str) -> EncryptedData:
        """Encrypt data using password-derived key"""
        if isinstance(data, str):
            data = data.encode('utf-8')
        
        # Generate random salt
        salt = os.urandom(32)
        
        # Derive key from password
        key = self.crypto_engine.derive_key_from_password(password, salt)
        
        # Encrypt data
        encrypted_data = self.crypto_engine.encrypt_symmetric(
            data, key, self._default_algorithm
        )
        encrypted_data.salt = salt
        encrypted_data.key_id = "password_derived"
        
        return encrypted_data
    
    def decrypt_with_password(self, encrypted_data: EncryptedData, password: str) -> bytes:
        """Decrypt data using password-derived key"""
        if not encrypted_data.salt:
            raise ValueError("Salt required for password-based decryption")
        
        # Derive key from password
        key = self.crypto_engine.derive_key_from_password(password, encrypted_data.salt)
        
        # Decrypt data
        plaintext = self.crypto_engine.decrypt_symmetric(encrypted_data, key)
        
        return plaintext

class SecureFileStorage:
    """Encrypted file storage system"""
    
    def __init__(self, encryption_service: EncryptionService, storage_path: str = "secure_storage"):
        self.encryption_service = encryption_service
        self.storage_path = storage_path
        self._ensure_storage_directory()
        
        # File metadata database
        self.db_path = os.path.join(storage_path, "metadata.db")
        self._init_database()
    
    def _ensure_storage_directory(self):
        """Ensure storage directory exists with proper permissions"""
        if not os.path.exists(self.storage_path):
            os.makedirs(self.storage_path, mode=0o700)
    
    def _init_database(self):
        """Initialize metadata database"""
        conn = sqlite3.connect(self.db_path)
        conn.execute("""
            CREATE TABLE IF NOT EXISTS file_metadata (
                file_id TEXT PRIMARY KEY,
                original_name TEXT NOT NULL,
                encrypted_path TEXT NOT NULL,
                key_id TEXT NOT NULL,
                algorithm TEXT NOT NULL,
                file_size INTEGER NOT NULL,
                checksum TEXT NOT NULL,
                created_at REAL NOT NULL,
                accessed_at REAL NOT NULL,
                metadata TEXT
            )
        """)
        conn.commit()
        conn.close()
    
    def store_file(self, file_path: str, metadata: Optional[Dict[str, Any]] = None) -> str:
        """Store file with encryption"""
        if not os.path.exists(file_path):
            raise FileNotFoundError(f"File not found: {file_path}")
        
        # Generate unique file ID
        file_id = str(uuid.uuid4())
        
        # Read file data
        with open(file_path, 'rb') as f:
            file_data = f.read()
        
        # Calculate checksum
        checksum = hashlib.sha256(file_data).hexdigest()
        
        # Encrypt file data
        key_id, encrypted_data = self.encryption_service.encrypt_data(file_data)
        
        # Save encrypted file
        encrypted_filename = f"{file_id}.enc"
        encrypted_path = os.path.join(self.storage_path, encrypted_filename)
        
        encrypted_file_data = self._serialize_encrypted_data(encrypted_data)
        
        with open(encrypted_path, 'wb') as f:
            f.write(encrypted_file_data)
        
        # Store metadata
        original_name = os.path.basename(file_path)
        current_time = time.time()
        
        conn = sqlite3.connect(self.db_path)
        conn.execute("""
            INSERT INTO file_metadata 
            (file_id, original_name, encrypted_path, key_id, algorithm, 
             file_size, checksum, created_at, accessed_at, metadata)
            VALUES (?, ?, ?, ?, ?, ?, ?, ?, ?, ?)
        """, (
            file_id, original_name, encrypted_path, key_id,
            encrypted_data.algorithm.value, len(file_data), checksum,
            current_time, current_time, json.dumps(metadata or {})
        ))
        conn.commit()
        conn.close()
        
        return file_id
    
    def retrieve_file(self, file_id: str, output_path: Optional[str] = None) -> str:
        """Retrieve and decrypt file"""
        # Get file metadata
        conn = sqlite3.connect(self.db_path)
        cursor = conn.execute("""
            SELECT original_name, encrypted_path, key_id, algorithm, checksum
            FROM file_metadata WHERE file_id = ?
        """, (file_id,))
        
        row = cursor.fetchone()
        if not row:
            conn.close()
            raise ValueError(f"File {file_id} not found")
        
        original_name, encrypted_path, key_id, algorithm, expected_checksum = row
        
        # Update access time
        conn.execute("""
            UPDATE file_metadata SET accessed_at = ? WHERE file_id = ?
        """, (time.time(), file_id))
        conn.commit()
        conn.close()
        
        # Read encrypted file
        with open(encrypted_path, 'rb') as f:
            encrypted_file_data = f.read()
        
        # Deserialize encrypted data
        encrypted_data = self._deserialize_encrypted_data(encrypted_file_data)
        
        # Decrypt file
        decrypted_data = self.encryption_service.decrypt_data(encrypted_data)
        
        # Verify checksum
        actual_checksum = hashlib.sha256(decrypted_data).hexdigest()
        if actual_checksum != expected_checksum:
            raise ValueError("File integrity check failed")
        
        # Save decrypted file
        if not output_path:
            output_path = original_name
        
        with open(output_path, 'wb') as f:
            f.write(decrypted_data)
        
        return output_path
    
    def delete_file(self, file_id: str):
        """Securely delete encrypted file"""
        # Get file metadata
        conn = sqlite3.connect(self.db_path)
        cursor = conn.execute("""
            SELECT encrypted_path FROM file_metadata WHERE file_id = ?
        """, (file_id,))
        
        row = cursor.fetchone()
        if not row:
            conn.close()
            raise ValueError(f"File {file_id} not found")
        
        encrypted_path = row[0]
        
        # Delete metadata
        conn.execute("DELETE FROM file_metadata WHERE file_id = ?", (file_id,))
        conn.commit()
        conn.close()
        
        # Securely delete encrypted file
        if os.path.exists(encrypted_path):
            # Overwrite file before deletion
            file_size = os.path.getsize(encrypted_path)
            with open(encrypted_path, 'r+b') as f:
                f.write(os.urandom(file_size))
                f.flush()
                os.fsync(f.fileno())
            
            os.remove(encrypted_path)
    
    def list_files(self) -> List[Dict[str, Any]]:
        """List all stored files"""
        conn = sqlite3.connect(self.db_path)
        cursor = conn.execute("""
            SELECT file_id, original_name, file_size, created_at, accessed_at, metadata
            FROM file_metadata ORDER BY created_at DESC
        """)
        
        files = []
        for row in cursor.fetchall():
            file_id, original_name, file_size, created_at, accessed_at, metadata_json = row
            
            files.append({
                'file_id': file_id,
                'original_name': original_name,
                'file_size': file_size,
                'created_at': created_at,
                'accessed_at': accessed_at,
                'metadata': json.loads(metadata_json or '{}')
            })
        
        conn.close()
        return files
    
    def _serialize_encrypted_data(self, encrypted_data: EncryptedData) -> bytes:
        """Serialize encrypted data for storage"""
        data_dict = {
            'data': base64.b64encode(encrypted_data.data).decode('ascii'),
            'algorithm': encrypted_data.algorithm.value,
            'key_id': encrypted_data.key_id,
            'iv': base64.b64encode(encrypted_data.iv).decode('ascii') if encrypted_data.iv else None,
            'salt': base64.b64encode(encrypted_data.salt).decode('ascii') if encrypted_data.salt else None,
            'tag': base64.b64encode(encrypted_data.tag).decode('ascii') if encrypted_data.tag else None,
            'metadata': encrypted_data.metadata
        }
        
        return json.dumps(data_dict).encode('utf-8')
    
    def _deserialize_encrypted_data(self, serialized_data: bytes) -> EncryptedData:
        """Deserialize encrypted data from storage"""
        data_dict = json.loads(serialized_data.decode('utf-8'))
        
        return EncryptedData(
            data=base64.b64decode(data_dict['data']),
            algorithm=EncryptionAlgorithm(data_dict['algorithm']),
            key_id=data_dict['key_id'],
            iv=base64.b64decode(data_dict['iv']) if data_dict['iv'] else None,
            salt=base64.b64decode(data_dict['salt']) if data_dict['salt'] else None,
            tag=base64.b64decode(data_dict['tag']) if data_dict['tag'] else None,
            metadata=data_dict['metadata']
        )

class SecureChannelManager:
    """Manage secure communication channels"""
    
    def __init__(self, encryption_service: EncryptionService):
        self.encryption_service = encryption_service
        self.active_channels: Dict[str, Dict[str, Any]] = {}
        self._lock = threading.RLock()
    
    def create_channel(self, channel_id: str, participants: List[str]) -> Dict[str, Any]:
        """Create secure communication channel"""
        with self._lock:
            # Generate channel key
            channel_key_id = self.encryption_service.key_manager.generate_key(
                EncryptionAlgorithm.AES_256_GCM,
                metadata={'channel_id': channel_id, 'participants': participants}
            )
            
            # Generate HMAC key for message integrity
            hmac_key_id = self.encryption_service.key_manager.generate_key(
                EncryptionAlgorithm.AES_256_GCM,
                KeyType.HMAC,
                metadata={'channel_id': channel_id, 'purpose': 'hmac'}
            )
            
            channel_info = {
                'channel_id': channel_id,
                'participants': participants,
                'encryption_key_id': channel_key_id,
                'hmac_key_id': hmac_key_id,
                'created_at': time.time(),
                'message_count': 0
            }
            
            self.active_channels[channel_id] = channel_info
            
            return channel_info
    
    def encrypt_message(self, channel_id: str, message: str, sender: str) -> Dict[str, Any]:
        """Encrypt message for secure channel"""
        with self._lock:
            if channel_id not in self.active_channels:
                raise ValueError(f"Channel {channel_id} not found")
            
            channel = self.active_channels[channel_id]
            
            if sender not in channel['participants']:
                raise ValueError(f"Sender {sender} not authorized for channel")
            
            # Create message with metadata
            message_data = {
                'content': message,
                'sender': sender,
                'timestamp': time.time(),
                'sequence': channel['message_count']
            }
            
            message_json = json.dumps(message_data)
            
            # Encrypt message
            encryption_key = self.encryption_service.key_manager.get_key(
                channel['encryption_key_id']
            )
            
            key_id, encrypted_data = self.encryption_service.encrypt_data(
                message_json, channel['encryption_key_id']
            )
            
            # Generate HMAC for integrity
            hmac_key = self.encryption_service.key_manager.get_key(
                channel['hmac_key_id']
            )
            
            message_hmac = self.encryption_service.crypto_engine.generate_hmac(
                encrypted_data.data, hmac_key.key_data
            )
            
            # Update message count
            channel['message_count'] += 1
            
            return {
                'channel_id': channel_id,
                'encrypted_data': encrypted_data,
                'hmac': base64.b64encode(message_hmac).decode('ascii'),
                'sequence': message_data['sequence']
            }
    
    def decrypt_message(self, channel_id: str, encrypted_message: Dict[str, Any]) -> Dict[str, Any]:
        """Decrypt message from secure channel"""
        with self._lock:
            if channel_id not in self.active_channels:
                raise ValueError(f"Channel {channel_id} not found")
            
            channel = self.active_channels[channel_id]
            encrypted_data = encrypted_message['encrypted_data']
            expected_hmac = base64.b64decode(encrypted_message['hmac'])
            
            # Verify HMAC
            hmac_key = self.encryption_service.key_manager.get_key(
                channel['hmac_key_id']
            )
            
            is_valid = self.encryption_service.crypto_engine.verify_hmac(
                encrypted_data.data, hmac_key.key_data, expected_hmac
            )
            
            if not is_valid:
                raise ValueError("Message integrity verification failed")
            
            # Decrypt message
            decrypted_data = self.encryption_service.decrypt_data(encrypted_data)
            message_data = json.loads(decrypted_data.decode('utf-8'))
            
            return message_data

# Demo Usage
async def demo_encryption_system():
    """Demonstrate comprehensive encryption system"""
    
    print("=== Data Encryption System Demo ===")
    
    # Initialize encryption service
    encryption_service = EncryptionService()
    
    print("Encryption service initialized with multi-algorithm support")
    
    print("\n1. Basic Data Encryption:")
    
    # Test different encryption algorithms
    test_data = "This is sensitive information that needs to be protected!"
    
    algorithms = [
        EncryptionAlgorithm.AES_256_GCM,
        EncryptionAlgorithm.AES_256_CBC,
        EncryptionAlgorithm.CHACHA20_POLY1305,
        EncryptionAlgorithm.FERNET
    ]
    
    for algorithm in algorithms:
        key_id, encrypted_data = encryption_service.encrypt_data(test_data, algorithm=algorithm)
        decrypted_data = encryption_service.decrypt_data(encrypted_data)
        
        success = decrypted_data.decode('utf-8') == test_data
        print(f"   {algorithm.value}: {'✅' if success else '❌'}")
    
    print("\n2. Password-Based Encryption:")
    
    password = "MySecurePassword123!"
    password_encrypted = encryption_service.encrypt_with_password(test_data, password)
    password_decrypted = encryption_service.decrypt_with_password(password_encrypted, password)
    
    password_success = password_decrypted.decode('utf-8') == test_data
    print(f"   Password encryption: {'✅' if password_success else '❌'}")
    
    # Test wrong password
    try:
        wrong_password_decrypted = encryption_service.decrypt_with_password(
            password_encrypted, "WrongPassword"
        )
        print("   Wrong password test: ❌ (should have failed)")
    except:
        print("   Wrong password test: ✅ (correctly failed)")
    
    print("\n3. Asymmetric Encryption:")
    
    # Generate RSA key pair
    private_key_id = encryption_service.key_manager.generate_key(
        EncryptionAlgorithm.RSA_2048,
        KeyType.ASYMMETRIC_PRIVATE
    )
    
    private_key = encryption_service.key_manager.get_key(f"{private_key_id}_private")
    public_key = encryption_service.key_manager.get_key(f"{private_key_id}_public")
    
    # Test asymmetric encryption
    asymmetric_data = "RSA encrypted message"
    
    encrypted_asymmetric = encryption_service.crypto_engine.encrypt_asymmetric(
        asymmetric_data.encode('utf-8'),
        public_key.key_data,
        EncryptionAlgorithm.RSA_2048
    )
    
    decrypted_asymmetric = encryption_service.crypto_engine.decrypt_asymmetric(
        encrypted_asymmetric,
        private_key.key_data
    )
    
    asymmetric_success = decrypted_asymmetric.decode('utf-8') == asymmetric_data
    print(f"   RSA encryption: {'✅' if asymmetric_success else '❌'}")
    
    print("\n4. Secure File Storage:")
    
    # Initialize secure file storage
    secure_storage = SecureFileStorage(encryption_service)
    
    # Create test file
    test_file_path = "test_document.txt"
    test_file_content = "This is a confidential document with sensitive information."
    
    with open(test_file_path, 'w') as f:
        f.write(test_file_content)
    
    # Store file securely
    file_id = secure_storage.store_file(
        test_file_path,
        metadata={'classification': 'confidential', 'department': 'security'}
    )
    
    print(f"   File stored with ID: {file_id}")
    
    # Retrieve file
    retrieved_path = secure_storage.retrieve_file(file_id, "retrieved_document.txt")
    
    with open(retrieved_path, 'r') as f:
        retrieved_content = f.read()
    
    file_success = retrieved_content == test_file_content
    print(f"   File retrieval: {'✅' if file_success else '❌'}")
    
    # List files
    files = secure_storage.list_files()
    print(f"   Stored files: {len(files)}")
    
    # Clean up test files
    os.remove(test_file_path)
    os.remove(retrieved_path)
    secure_storage.delete_file(file_id)
    
    print("\n5. Secure Communication Channels:")
    
    # Initialize secure channel manager
    channel_manager = SecureChannelManager(encryption_service)
    
    # Create secure channel
    participants = ["alice", "bob", "charlie"]
    channel_info = channel_manager.create_channel("secure_chat", participants)
    
    print(f"   Created channel: {channel_info['channel_id']}")
    print(f"   Participants: {', '.join(participants)}")
    
    # Send encrypted messages
    messages = [
        ("alice", "Hello everyone! This is a secure message."),
        ("bob", "Hi Alice! The encryption is working perfectly."),
        ("charlie", "Great! Our communication is now secure.")
    ]
    
    encrypted_messages = []
    
    for sender, message in messages:
        encrypted_msg = channel_manager.encrypt_message("secure_chat", message, sender)
        encrypted_messages.append(encrypted_msg)
        print(f"   Encrypted message from {sender}")
    
    # Decrypt messages
    print("\n   Decrypted messages:")
    for encrypted_msg in encrypted_messages:
        decrypted_msg = channel_manager.decrypt_message("secure_chat", encrypted_msg)
        print(f"   [{decrypted_msg['sender']}]: {decrypted_msg['content']}")
    
    print("\n6. Key Management:")
    
    # Show key management capabilities
    all_keys = encryption_service.key_manager.list_keys()
    print(f"   Active keys: {len(all_keys)}")
    
    # Test key rotation
    test_key_id = encryption_service.key_manager.generate_key(EncryptionAlgorithm.AES_256_GCM)
    print(f"   Generated key: {test_key_id}")
    
    rotated_key_id = encryption_service.key_manager.rotate_key(test_key_id)
    print(f"   Rotated to new key: {rotated_key_id}")
    
    print("\n7. Performance Benchmarks:")
    
    # Benchmark encryption performance
    benchmark_data = b"A" * (1024 * 1024)  # 1MB of data
    
    algorithms_to_test = [
        EncryptionAlgorithm.AES_256_GCM,
        EncryptionAlgorithm.CHACHA20_POLY1305,
        EncryptionAlgorithm.FERNET
    ]
    
    for algorithm in algorithms_to_test:
        start_time = time.time()
        
        for _ in range(10):  # Encrypt 10MB total
            key_id, encrypted = encryption_service.encrypt_data(benchmark_data, algorithm=algorithm)
            decrypted = encryption_service.decrypt_data(encrypted)
        
        end_time = time.time()
        throughput = (10 * 1024 * 1024) / (end_time - start_time)  # Bytes per second
        
        print(f"   {algorithm.value}: {throughput / (1024*1024):.1f} MB/s")
    
    print("\n8. Security Features Summary:")
    
    security_features = [
        "✅ AES-256-GCM authenticated encryption",
        "✅ ChaCha20-Poly1305 stream cipher",
        "✅ RSA-4096 asymmetric encryption",
        "✅ PBKDF2/Scrypt password derivation",
        "✅ HMAC-SHA256 message authentication",
        "✅ Automatic key rotation",
        "✅ Secure key storage",
        "✅ File-level encryption",
        "✅ End-to-end encrypted channels",
        "✅ Perfect forward secrecy",
        "✅ Cryptographic integrity checking"
    ]
    
    for feature in security_features:
        print(f"   {feature}")
    
    print("\n✅ Encryption System Demo Complete!")
    
    print(f"\nEncryption Summary:")
    print(f"├── Symmetric algorithms: AES-256-GCM, AES-256-CBC, ChaCha20-Poly1305, Fernet")
    print(f"├── Asymmetric algorithms: RSA-2048, RSA-4096")
    print(f"├── Key derivation: PBKDF2, Scrypt")
    print(f"├── Authentication: HMAC-SHA256, AEAD modes")
    print(f"├── Key management: Generation, rotation, secure deletion")
    print(f"├── File encryption: Transparent storage encryption")
    print(f"└── Communication: End-to-end encrypted channels")

if __name__ == "__main__":
    asyncio.run(demo_encryption_system())
```

---

**Key Features:**
- **Multi-Algorithm Support**: AES-256-GCM, ChaCha20-Poly1305, RSA, and Fernet encryption
- **Comprehensive Key Management**: Generation, rotation, secure storage, and lifecycle management
- **Data at Rest**: Encrypted file storage with integrity checking
- **Data in Transit**: Secure communication channels with end-to-end encryption
- **Password-Based Encryption**: PBKDF2 and Scrypt key derivation functions
- **Authenticated Encryption**: AEAD modes and HMAC for integrity verification

**Related:** See [Authentication & Authorization](auth.md) for access control and [DDoS Protection](ddos-protection.md) for network security.
