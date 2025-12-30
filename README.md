StockFlow – Backend Engineering Intern Case Study

Name: Abhay Phutela

Overview

The given API endpoint is responsible for creating a product and initializing its inventory for a warehouse. Although the code compiles, it fails to meet production-grade and business requirements for a B2B inventory management system.

 *** Issues Identified

1 No input validation
2 SKU uniqueness not enforced
3 Incorrect data modeling
4 lack of transactional integrity
5 Unsafe price handling
6 Optional fields not handled
7 Fields like initial_quantity may be missing.
8 No error handling or rollback

2. Impact in Production

API crashes and 500 errors
Duplicate SKUs causing incorrect order fulfillment
Orphan products without inventory
Incorrect financial calculations
Inconsistent stock data across warehouses
Difficult debugging and maintenance


*####Corrected Implementation (Java – Spring Boot)

Although the original code was provided in Flask (Python), I’ve implemented the corrected solution in Java + Spring Boot to demonstrate backend principles using my primary production stack.

Key Fixes

Product made warehouse-independent
Inventory handles product-warehouse relationship
Transactional safety added
SKU uniqueness enforced
BigDecimal used for price
Optional fields handled safely


@RestController
@RequestMapping("/api/products")
@RequiredArgsConstructor
public class ProductController {

    private final ProductService productService;

    @PostMapping
    public ResponseEntity<?> createProduct(
            @Valid @RequestBody CreateProductRequest request) {

        Long productId = productService.createProduct(request);

        return ResponseEntity.status(HttpStatus.CREATED)
                .body(Map.of(
                    "message", "Product created successfully",
                    "productId", productId
                ));
    }
}

@Service
@RequiredArgsConstructor
public class ProductService {

    private final ProductRepository productRepository;
    private final InventoryRepository inventoryRepository;

    @Transactional
    public Long createProduct(CreateProductRequest request) {

        if (productRepository.existsBySku(request.getSku())) {
            throw new IllegalArgumentException("SKU already exists");
        }

        Product product = new Product();
        product.setName(request.getName());
        product.setSku(request.getSku());
        product.setPrice(request.getPrice());

        product = productRepository.save(product);

        Inventory inventory = new Inventory();
        inventory.setProduct(product);
        inventory.setWarehouseId(request.getWarehouseId());
        inventory.setQuantity(
            request.getInitialQuantity() != null ? request.getInitialQuantity() : 0
        );

        inventoryRepository.save(inventory);

        return product.getId();
    }
}
