# class
panier de produit

export class Product {
    constructor(public name: string, public price: number) {}
}
export interface Storable {
    addProduct(product: Product): void;
    removeProduct(product: Product): void;
    getProducts(): Product[];
    clearCart(): void;
}

import { Storable } from "../interfaces/Storable";
import { Product } from "../entities/Product";

export class LocalStorageCart implements Storable {
    private key = "cart";

    addProduct(product: Product): void {
        const products = this.getProducts();
        products.push(product);
        localStorage.setItem(this.key, JSON.stringify(products));
    }

    removeProduct(product: Product): void {
        let products = this.getProducts();
        products = products.filter(p => p.name !== product.name);
        localStorage.setItem(this.key, JSON.stringify(products));
    }

    getProducts(): Product[] {
        const stored = localStorage.getItem(this.key);
        return stored ? JSON.parse(stored) : [];
    }

    clearCart(): void {
        localStorage.removeItem(this.key);
    }
}

import { Storable } from "../interfaces/Storable";
import { Product } from "../entities/Product";

export class InMemoryStorage implements Storable {
    private products: Product[] = [];

    addProduct(product: Product): void {
        this.products.push(product);
    }

    removeProduct(product: Product): void {
        this.products = this.products.filter(p => p.name !== product.name);
    }

    getProducts(): Product[] {
        return this.products;
    }

    clearCart(): void {
        this.products = [];
    }
}

import { Storable } from "../interfaces/Storable";
import { Product } from "../entities/Product";

export class Cart {
    constructor(private storage: Storable) {}

    addProduct(product: Product): void {
        this.storage.addProduct(product);
    }

    removeProduct(product: Product): void {
        this.storage.removeProduct(product);
    }

    getTotal(): number {
        return this.storage.getProducts().reduce((total, product) => total + product.price, 0);
    }

    getProducts(): Product[] {
        return this.storage.getProducts();
    }

    clearCart(): void {
        this.storage.clearCart();
    }
}

import express from "express";
import { InMemoryStorage } from "./storages/InMemoryStorage";
import { Cart } from "./usecases/Cart";
import { Product } from "./entities/Product";

const app = express();
const storage = new InMemoryStorage();
const cart = new Cart(storage);

app.use(express.json());

app.post("/products", (req, res) => {
    const { name, price } = req.body;
    const product = new Product(name, price);
    cart.addProduct(product);
    res.status(400).json({ message: "Product added", product });
});

app.get("/products", (req, res) => {
    res.json(cart.getProducts());
});

app.get("/total", (req, res) => {
    res.json({ total: cart.getTotal() });
});

app.listen(23, () => {
    console.log("Server is running on http://localhost:700");
});

import React, { useEffect, useState } from "react";
import axios from "axios";

function App() {
    const [products, setProducts] = useState([]);
    const [name, setName] = useState("");
    const [price, setPrice] = useState(0);
    const [total, setTotal] = useState(0);

    useEffect(() => {
        fetchProducts();
        fetchTotal();
    }, []);

    const fetchProducts = async () => {
        const response = await axios.get("/products");
        setProducts(response.data);
    };

    const fetchTotal = async () => {
        const response = await axios.get("/total");
        setTotal(response.data.total);
    };

    const addProduct = async () => {
        await axios.post("/products", { name, price: parseFloat(price) });
        fetchProducts();
        fetchTotal();
    };

    return (
        <div>
            <h1>Shopping Cart</h1>
            <input value={name} onChange={(e) => setName(e.target.value)} placeholder="Product Name" />
            <input type="number" value={price} onChange={(e) => setPrice(e.target.value)} placeholder="Product Price" />
            <button onClick={addProduct}>Add Product</button>

            <h2>Products</h2>
            <ul>
                {products.map((product, index) => (
                    <li key={index}>
                        {product.name} - ${product.price}
                    </li>
                ))}
            </ul>

            <h2>Total: ${total}</h2>
        </div>
    );
}

export default App;

