# MARKDOWN-patron-repository
Ejemplo del Patrón Repository

Entiendo que en aplicaciones simples o en etapas tempranas del desarrollo, el patrón de repositorio puede parecer redundante. Sin embargo, en sistemas más complejos, su utilidad se vuelve mucho más evidente. Aquí te doy un ejemplo donde el patrón de repositorio no solo no es redundante, sino que resulta esencial.

### **Ejemplo Complejo: Aplicación de E-commerce**

Imagina que estás desarrollando una aplicación de e-commerce donde gestionas productos, usuarios, órdenes de compra, etc. En este sistema, podrías tener múltiples fuentes de datos y lógica de negocio compleja.

#### **Requisitos del Sistema:**
- **Productos**: Los productos se almacenan en una base de datos SQL.
- **Inventario**: La información de inventario se obtiene de un sistema externo vía una API REST.
- **Precios**: Los precios se calculan dinámicamente basados en reglas de negocio que involucran descuentos, impuestos, etc.

#### **Arquitectura con el Patrón Repository**

1. **DataSource Implementations**:
   - **ProductDatasource**: Interactúa con la base de datos SQL para CRUD de productos.
   - **InventoryDatasource**: Hace peticiones a la API REST externa para obtener la información del inventario.
   - **PricingDatasource**: Calcula el precio basado en reglas complejas, combinando datos de múltiples fuentes.

   ```typescript
   export abstract class ProductDatasource {
       abstract createProduct(data: CreateProductDto): Promise<ProductEntity>;
       abstract findProductById(id: number): Promise<ProductEntity>;
       // Otros métodos...
   }

   export abstract class InventoryDatasource {
       abstract getInventoryStatus(productId: number): Promise<InventoryStatus>;
       // Otros métodos...
   }

   export abstract class PricingDatasource {
       abstract calculatePrice(productId: number): Promise<number>;
       // Otros métodos...
   }
   ```

2. **Repository Implementation**:
   - **ProductRepository**: Este repositorio centraliza la lógica de negocio relacionada con los productos. No solo interactúa con el `ProductDatasource`, sino también con el `InventoryDatasource` y el `PricingDatasource`.

   ```typescript
   export class ProductRepository {
       constructor(
           private readonly productDatasource: ProductDatasource,
           private readonly inventoryDatasource: InventoryDatasource,
           private readonly pricingDatasource: PricingDatasource,
       ) {}

       async createProduct(data: CreateProductDto): Promise<ProductEntity> {
           // 1. Crea el producto en la base de datos.
           const product = await this.productDatasource.createProduct(data);

           // 2. Calcula el precio usando el PricingDatasource.
           const price = await this.pricingDatasource.calculatePrice(product.id);
           product.setPrice(price);

           // 3. Obtén el estado del inventario usando el InventoryDatasource.
           const inventoryStatus = await this.inventoryDatasource.getInventoryStatus(product.id);
           product.setInventoryStatus(inventoryStatus);

           // 4. Devuelve el producto con toda la información compilada.
           return product;
       }

       async getProductDetails(id: number): Promise<ProductEntity> {
           const product = await this.productDatasource.findProductById(id);
           const price = await this.pricingDatasource.calculatePrice(product.id);
           const inventoryStatus = await this.inventoryDatasource.getInventoryStatus(product.id);

           product.setPrice(price);
           product.setInventoryStatus(inventoryStatus);

           return product;
       }

       // Otros métodos...
   }
   ```

3. **Uso en el Controlador**:
   - En el controlador, solo interactúas con el repositorio, sin preocuparte de los detalles de cómo se obtienen o calculan los datos.

   ```typescript
   export class ProductController {
       constructor(private readonly productRepository: ProductRepository) {}

       async createProduct(req: Request, res: Response) {
           const productDto = req.body;
           const product = await this.productRepository.createProduct(productDto);
           return res.json(product);
       }

       async getProduct(req: Request, res: Response) {
           const productId = +req.params.id;
           const product = await this.productRepository.getProductDetails(productId);
           return res.json(product);
       }

       // Otros métodos...
   }
   ```

### **¿Por qué No Es Redundante?**

- **Centralización de la Lógica de Negocio**: La lógica para manejar productos, precios, e inventario se centraliza en el repositorio. Esto asegura que cualquier cambio en la lógica de negocio se haga en un solo lugar.

- **Uso de Múltiples DataSources**: El repositorio interactúa con múltiples fuentes de datos. Si no tuvieras el repositorio, el controlador tendría que gestionar la lógica de cómo combinar estas fuentes, lo que rompería la separación de responsabilidades.

- **Flexibilidad**: Si en el futuro decides cambiar la fuente de datos para los productos o modificar las reglas de precios, solo necesitas cambiar la implementación del DataSource o la lógica dentro del repositorio, sin afectar el resto del código.

### **Conclusión**

En aplicaciones simples, el patrón de repositorio puede parecer un paso innecesario. Sin embargo, a medida que la complejidad del sistema crece, la necesidad de una capa que centralice y gestione la lógica de negocio y la interacción con múltiples fuentes de datos se vuelve crucial. En este ejemplo de e-commerce, el patrón de repositorio permite mantener el código limpio, modular, y fácil de mantener y extender.

