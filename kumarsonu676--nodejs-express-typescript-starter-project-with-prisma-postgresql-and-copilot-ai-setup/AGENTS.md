# Copilot Instructions for the NodeJs Express Project

## Folder and File Structure Guidelines

1. **Public**: Place all static files in the `public` folder.
2. **Src**: Place all source code in the `src` folder.
   - **Controllers**: Place all route controllers in the `src/controllers` folder.
   - **Middlewares**: Place all middleware functions in the `src/middlewares` folder.
   - **Models**: Place all data models in the `src/models` folder.
   - **Services**: Place all business logic and service files in the `src/services` folder.
   - **Utils**: Place utility functions in the `src/utils` folder.
   - **Routes**: Place all route definitions in the `src/routes` folder.
   - **Config**: Place configuration files in the `src/config` folder.
   - **DTOs**: Place dto files in the `src/dtos` folder.
   - **Validation**: Place Yup validation files in the `src/validators` folder.
   - **Parameters**: Place filter parameters files in the `src/params` folder.
   - **Repositories**: Place repositories files in the `src/repositories` folder.

## Coding Guidelines

- Use **ES6+** syntax (e.g., `const/let`, arrow functions, `async/await`).
- **Validate incoming request data** using Yup schemas before reaching controllers.
- **Keep controllers thin** by delegating business logic to services.
- **Handle errors** gracefully using centralized error-handling middleware.
- **Organize code** with clean imports and avoid circular dependencies.
- **Naming conventions**:
  - camelCase for variables, functions, and file names.
  - PascalCase for classes and DTOs.
- Write **modular, reusable, and well-documented** code.
- When creating a new route or controller, always create a corresponding DTO and validation schema.

## Special Instructions for Copilot

- Only generate code that strictly follows this folder structure and coding style.
- Always separate concerns: controllers should not directly contain business logic or validation.
- Ensure error messages are meaningful and consistent across all APIs.
- Prioritize clean and maintainable code over shortcuts or hacks.

## Best Practices for Node.js Express with TypeScript

1. **Use TypeScript**: Always define types and interfaces for request, response, and other objects.
2. **Modular Structure**: Separate concerns by organizing code into controllers, services, and routes.
3. **Error Handling**: Use middleware for centralized error handling.
4. **Environment Variables**: Store sensitive data in `.env` files and access them using libraries like `dotenv`.
5. **Validation**: Use library `Yup` to validate request payloads.
6. **Logging**: Use a logging library `winston` for structured logging.
7. **Security**: Use middleware like `helmet` and `cors` to enhance security.
8. **Testing**: Write unit and integration tests using frameworks like `Jest` or `Mocha`.

---

## Code Generation Examples

### Example: Creating a Controller

```ts
// filepath: src/controllers/example.controller.ts
import { Request, Response } from 'express';
import CustomResponse from '../dtos/custom-response';
import { UpdateUserDto, UserDto } from '../dtos/user.dto';
import { CreateUserModel } from '../models/user.model';
import CustomError from '../exceptions/custom-error';
import container from '../config/ioc.config';
import IUnitOfService from '../services/interfaces/iunitof.service';
import { TYPES } from '../config/ioc.types';

export class UserController {
  constructor(private unitOfService = container.get<IUnitOfService>(TYPES.IUnitOfService)) {
    this.unitOfService = unitOfService;
  }

  getUserById = async (req: Request, res: Response): Promise<Response<CustomResponse<UserDto>>> => {
    const userId = req.params.id;
    const user = await this.unitOfService.User.findById(userId);
    if (!user) {
      return res.status(404).json({ message: 'User not found' });
    }

    const response: CustomResponse<UserDto> = {
      success: true,
      data: user,
    };

    return res.status(200).json(response);
  };

  getUserByEmail = async (req: Request, res: Response): Promise<Response<CustomResponse<UserDto>>> => {
    const email = req.query.email as string;
    if (!email) {
      return res.status(400).json({ message: 'Email is required' });
    }

    const user = await this.unitOfService.User.findByEmail(email, false);
    if (!user) {
      return res.status(404).json({ message: 'User not found' });
    }

    const response: CustomResponse<UserDto> = {
      success: true,
      data: user,
    };

    return res.status(200).json(response);
  };

  createUser = async (req: Request, res: Response): Promise<Response<CustomResponse<UserDto>>> => {
    const data = req.body as CreateUserModel;

    let user = await this.unitOfService.User.findByEmail(data.email, false);
    if (user) {
      throw new CustomError('User already exists', 409);
    }

    user = await this.unitOfService.User.create(data);

    let response: CustomResponse<UserDto>;

    if (!user) {
      response = {
        success: false,
        message: 'User creation failed',
      };
      return res.status(400).json(response);
    }

    response = {
      success: true,
      data: user,
    };

    return res.status(201).json(response);
  };

  updateUserById = async (req: Request, res: Response): Promise<Response<CustomResponse<UserDto>>> => {
    const userId = req.params.id;
    const data = req.body as UpdateUserDto;
    const user = await this.unitOfService.User.update(userId, data);

    if (!user) {
      return res.status(404).json({ message: 'User not found' });
    }

    const response: CustomResponse<UserDto> = {
      success: true,
      data: user,
    };

    return res.status(200).json(response);
  };

  deleteUserById = async (req: Request, res: Response): Promise<Response<CustomResponse<UserDto>>> => {
    const userId = req.params.id;
    const user = await this.unitOfService.User.delete(userId);
    if (!user) {
      return res.status(404).json({ message: 'User not found' });
    }

    const response: CustomResponse<UserDto> = {
      success: true,
      data: user,
    };

    return res.status(200).json(response);
  };
}
```

### Example: Creating a Route

```ts
// filepath: src/routes/example.routes.ts
import { Router } from 'express';
import asyncHandler from '../middlewares/asyncHandler.middleware';
import validateSchema from '../middlewares/validation.middleware';
import CreateUserValidator from '../validators/user.validator';
import { UserController } from '../controllers/user.controller';
import { TYPES } from '../config/ioc.types';
import container from '../config/ioc.config';

const userRouter = Router();

const userController = container.get<UserController>(TYPES.UserController);

userRouter.get('/getbyemail', asyncHandler(userController.getUserByEmail));
userRouter.get('/:id', asyncHandler(userController.getUserById));
userRouter.post('/', [validateSchema(CreateUserValidator)], asyncHandler(userController.createUser));
userRouter.put('/:id', asyncHandler(userController.updateUserById));
userRouter.delete('/:id', asyncHandler(userController.deleteUserById));

export default userRouter;
```

### After creating route, Add in base routing in "src/routes/index.routes.ts" file

```ts
// filepath: src/routes/index.routes.ts
import exampleRouter from './example.routes';

routes.use('/example', exampleRouter);
```

### Example: Creating a Middleware

```ts
// filepath: src/middlewares/logger.middleware.ts
import { Request, Response, NextFunction } from 'express';
import CustomResponse from '../dtos/custom-response';
import PlainDto from '../dtos/plain.dto';

class ClientIdMiddleware {
  verify(req: Request, res: Response, next: NextFunction) {
    const clientId: string = Array.isArray(req.headers['clientid']) ? req.headers['clientid'][0] : req.headers['clientid'] || '';
    const _clientId = process.env.CLIENT_ID;

    if (!clientId && process.env.SITE_MODE !== 'local') {
      const response: CustomResponse<PlainDto> = {
        success: false,
        message: 'ClientId header is missing',
      };
      res.status(401).json(response);
      return;
    }

    if (clientId !== _clientId && process.env.SITE_MODE !== 'local') {
      const response: CustomResponse<PlainDto> = {
        success: false,
        message: 'Invalid Client Id',
      };
      return res.status(401).json(response);
    }

    req.headers['clientid'] = clientId;

    // Call next to pass control to the next middleware or route handler
    next();
  }
}

export default new ClientIdMiddleware();
```

### Example: Creating a Repository

- Repository Interface

```ts
// filepath: src/repositories/interfaces/iexample.repository.ts
import { CreateUserDto, UpdateUserDto } from '../../dtos/user.dto';
import { UserFilterParams } from '../../params/user.params';
import { AuthProvider, User, UserRole } from '../../prisma/generated';

export interface IUserRepository {
  findAll(
    filters?: UserFilterParams,
    page?: number,
    limit?: number,
    sortBy?: string,
    sortOrder?: string
  ): Promise<{
    users: User[];
    total: number;
    page: number;
    limit: number;
    totalPages: number;
  }>;
  findById(id: string): Promise<User | null>;
  findByEmail(email: string): Promise<User | null>;
  findByPhoneNumber(phoneNumber: string): Promise<User | null>;
  findByProviderId(providerId: string, provider: AuthProvider): Promise<User | null>;
  create(data: CreateUserDto): Promise<User>;
  update(id: string, data: UpdateUserDto): Promise<User>;
  delete(id: string): Promise<User>;
  updatePassword(id: string, passwordHash: string): Promise<User>;
  verifyEmail(id: string): Promise<User>;
  setTwoFactorEnabled(id: string, enabled: boolean, secret?: string): Promise<User>;
  updateRole(id: string, role: UserRole): Promise<User>;
  setActiveStatus(id: string, isActive: boolean): Promise<User>;
  countByRole(): Promise<{ role: UserRole; count: number }[]>;
  findByDateRange(startDate: Date, endDate: Date, page: number, limit: number): Promise<{ users: User[]; total: number }>;
}
```

- Repository Class

```ts
// filepath: src/repositories/example.repository.ts
import prisma from '../prisma';
import { CreateUserDto, UpdateUserDto } from '../dtos/user.dto';
import { UserFilterParams } from '../params/user.params';
import { AuthProvider, Prisma, User, UserRole } from '../prisma/generated';
import { IUserRepository } from './interfaces/iuser.repository';

export class UserRepository implements IUserRepository {
  /**
   * Find a user by their unique ID
   */
  async findById(id: string): Promise<User | null> {
    return prisma.user.findUnique({
      where: { id },
    });
  }

  /**
   * Find a user by their email address
   */
  async findByEmail(email: string): Promise<User | null> {
    return prisma.user.findUnique({
      where: { email },
    });
  }

  /**
   * Find a user by their phone number
   */
  async findByPhoneNumber(phoneNumber: string): Promise<User | null> {
    return prisma.user.findUnique({
      where: { phoneNumber },
    });
  }

  /**
   * Find a user by their provider ID
   */
  async findByProviderId(providerId: string, provider: AuthProvider): Promise<User | null> {
    return prisma.user.findFirst({
      where: {
        providerId,
        provider,
      },
    });
  }

  /**
   * Create a new user
   */
  async create(data: CreateUserDto): Promise<User> {
    return prisma.user.create({
      data,
    });
  }

  /**
   * Update an existing user
   */
  async update(id: string, data: UpdateUserDto): Promise<User> {
    return prisma.user.update({
      where: { id },
      data,
    });
  }

  /**
   * Delete a user by ID
   */
  async delete(id: string): Promise<User> {
    return prisma.user.delete({
      where: { id },
    });
  }

  /**
   * Find all users with optional filtering, pagination and sorting
   */
  async findAll(
    filters?: UserFilterParams,
    page = 1,
    limit = 10,
    sortBy = 'createdAt',
    sortOrder: 'asc' | 'desc' = 'desc'
  ): Promise<{
    users: User[];
    total: number;
    page: number;
    limit: number;
    totalPages: number;
  }> {
    // Build where clause based on filters
    const where: Prisma.UserWhereInput = {};

    if (filters) {
      // Add each filter condition if provided
      if (filters.email) where.email = { contains: filters.email, mode: 'insensitive' };
      if (filters.phoneNumber) where.phoneNumber = { contains: filters.phoneNumber };
      if (filters.role) where.role = filters.role;
      if (filters.isActive !== undefined) where.isActive = filters.isActive;
      if (filters.provider) where.provider = filters.provider;
      if (filters.isEmailVerified !== undefined) where.isEmailVerified = filters.isEmailVerified;
      if (filters.isPhoneVerified !== undefined) where.isPhoneVerified = filters.isPhoneVerified;
      if (filters.twoFactorEnabled !== undefined) where.twoFactorEnabled = filters.twoFactorEnabled;

      // Handle firstName and lastName filters
      if (filters.firstName) where.firstName = { contains: filters.firstName, mode: 'insensitive' };
      if (filters.lastName) where.lastName = { contains: filters.lastName, mode: 'insensitive' };

      // Handle search across multiple fields
      if (filters.search) {
        where.OR = [
          { firstName: { contains: filters.search, mode: 'insensitive' } },
          { lastName: { contains: filters.search, mode: 'insensitive' } },
          { email: { contains: filters.search, mode: 'insensitive' } },
        ];
      }
    }

    // Calculate skip value for pagination
    const skip = (page - 1) * limit;

    // Get total count for pagination info
    const total = await prisma.user.count({ where });

    // Get users with pagination and sorting
    const users = await prisma.user.findMany({
      where,
      skip,
      take: limit,
      orderBy: {
        [sortBy]: sortOrder,
      },
    });

    return {
      users,
      total,
      page,
      limit,
      totalPages: Math.ceil(total / limit),
    };
  }

  /**
   * Update user password
   */
  async updatePassword(id: string, passwordHash: string): Promise<User> {
    return prisma.user.update({
      where: { id },
      data: { passwordHash },
    });
  }

  /**
   * Verify user email
   */
  async verifyEmail(id: string): Promise<User> {
    return prisma.user.update({
      where: { id },
      data: { isEmailVerified: true },
    });
  }

  /**
   * Verify user phone
   */
  async verifyPhone(id: string): Promise<User> {
    return prisma.user.update({
      where: { id },
      data: { isPhoneVerified: true },
    });
  }

  /**
   * Enable or disable two-factor authentication
   */
  async setTwoFactorEnabled(id: string, enabled: boolean, secret?: string): Promise<User> {
    return prisma.user.update({
      where: { id },
      data: {
        twoFactorEnabled: enabled,
        twoFactorSecret: enabled ? secret : null,
      },
    });
  }

  /**
   * Update user role
   */
  async updateRole(id: string, role: UserRole): Promise<User> {
    return prisma.user.update({
      where: { id },
      data: { role },
    });
  }

  /**
   * Activate or deactivate a user
   */
  async setActiveStatus(id: string, isActive: boolean): Promise<User> {
    return prisma.user.update({
      where: { id },
      data: { isActive },
    });
  }

  /**
   * Count users by role
   */
  async countByRole(): Promise<{ role: UserRole; count: number }[]> {
    const results = await prisma.user.groupBy({
      by: ['role'],
      _count: {
        role: true,
      },
    });

    return results.map((result) => ({
      role: result.role,
      count: result._count.role,
    }));
  }

  /**
   * Find users created within a date range
   */
  async findByDateRange(startDate: Date, endDate: Date, page = 1, limit = 10): Promise<{ users: User[]; total: number }> {
    const skip = (page - 1) * limit;

    const where: Prisma.UserWhereInput = {
      createdAt: {
        gte: startDate,
        lte: endDate,
      },
    };

    const total = await prisma.user.count({ where });

    const users = await prisma.user.findMany({
      where,
      skip,
      take: limit,
      orderBy: {
        createdAt: 'desc',
      },
    });

    return { users, total };
  }
}
```

- Once Repository Interface and class is created add the reference in `iunitofwork.repository.ts` and `unitofwork.repository.ts` file

```ts
// filepath: src/repositories/interfaces/iunitofwork.repository.ts
import { IUserRepository } from './iuser.repository';

export default interface IUnitOfWork {
  User: IUserRepository;
}
```

```ts
// filepath: src/repositories/unitofwork.repository.ts
import container from '../config/ioc.config';
import { TYPES } from '../config/ioc.types';
import IUnitOfWork from './interfaces/iunitofwork.repository';
import { IUserRepository } from './interfaces/iuser.repository';

export default class UnitOfWork implements IUnitOfWork {
  public User: IUserRepository;
  constructor(user = container.get<IUserRepository>(TYPES.IUserRepository)) {
    this.User = user;
  }
}
```

### Example: Creating a Service

- Creating Service Interface

```ts
// filepath: src/services/interfaces/iexample.service.ts

import { UpdateUserDto, UserDto } from '../../dtos/user.dto';
import { CreateUserModel } from '../../models/user.model';
import { AuthProvider, User } from '../../prisma/generated';

export interface IUserService {
  findById(id: string): Promise<UserDto | null>;
  findByEmail(email: string, includePassword: boolean): Promise<UserDto | null>;
  findByPhoneNumber(phoneNumber: string): Promise<UserDto | null>;
  findByProviderId(providerId: string, provider: AuthProvider): Promise<UserDto | null>;
  create(data: CreateUserModel): Promise<UserDto | null>;
  update(id: string, data: UpdateUserDto): Promise<UserDto | null>;
  delete(id: string): Promise<UserDto | null>;
  convertToDto(user: User, includePassword: boolean): UserDto;
}
```

- Creating Service Class

```ts
// filepath: src/services/example.service.ts
import { inject, injectable } from 'inversify';
import { TYPES } from '../config/ioc.types';
import { UpdateUserDto, UserDto } from '../dtos/user.dto';
import { CreateUserModel } from '../models/user.model';
import { AuthProvider, User, UserRole } from '../prisma/generated';
import IUnitOfWork from '../repositories/interfaces/iunitofwork.repository';
import PasswordUtils from '../utils/password.utils';
import { IUserService } from './interfaces/iuser.service';

@injectable()
export class UserService implements IUserService {
  constructor(@inject(TYPES.IUnitOfWork) private unitOfWork: IUnitOfWork) {}

  async findById(id: string): Promise<UserDto | null> {
    const user = await this.unitOfWork.User.findById(id);
    if (!user) {
      return null;
    }
    return this.convertToDto(user);
  }

  async findByEmail(email: string, includePassword: boolean = false): Promise<UserDto | null> {
    const user = await this.unitOfWork.User.findByEmail(email);
    if (!user) {
      return null;
    }
    return this.convertToDto(user, includePassword);
  }

  async findByPhoneNumber(phoneNumber: string): Promise<UserDto | null> {
    const user = await this.unitOfWork.User.findByPhoneNumber(phoneNumber);
    if (!user) {
      return null;
    }
    return this.convertToDto(user);
  }

  async findByProviderId(providerId: string, provider: AuthProvider): Promise<UserDto | null> {
    const user = await this.unitOfWork.User.findByProviderId(providerId, provider);
    if (!user) {
      return null;
    }
    return this.convertToDto(user);
  }

  async create(data: CreateUserModel): Promise<UserDto | null> {
    const hashedPassword = await PasswordUtils.hashPassword(data.password);
    const user = await this.unitOfWork.User.create({
      email: data.email,
      passwordHash: hashedPassword,
      firstName: data.firstName,
      lastName: data.lastName,
      phoneNumber: data.phoneNumber,
      phoneCountryCode: data.phoneCountryCode,
      isEmailVerified: false,
      isPhoneVerified: false,
      twoFactorEnabled: false,
      role: UserRole.USER,
      provider: AuthProvider.CREDENTIALS,
      isActive: true,
    });
    return this.convertToDto(user);
  }

  async update(id: string, data: UpdateUserDto): Promise<UserDto | null> {
    const user = await this.unitOfWork.User.update(id, {
      ...data,
    });
    if (!user) {
      return null;
    }
    return this.convertToDto(user);
  }

  async delete(id: string): Promise<UserDto | null> {
    const user = await this.unitOfWork.User.delete(id);
    return this.convertToDto(user);
  }

  //convert User to UserDto
  convertToDto(user: User, includePassword: boolean = false): UserDto {
    return {
      id: user.id,
      email: user.email,
      passwordHash: includePassword ? user.passwordHash : undefined,
      firstName: user.firstName,
      lastName: user.lastName,
      phoneNumber: user.phoneNumber,
      phoneCountryCode: user.phoneCountryCode,
      dateOfBirth: user.dateOfBirth,
      profileImageUrl: user.profileImageUrl,
      provider: user.provider,
      providerId: user.providerId,
      isEmailVerified: user.isEmailVerified,
      isPhoneVerified: user.isPhoneVerified,
      twoFactorEnabled: user.twoFactorEnabled,
      twoFactorSecret: user.twoFactorSecret,
      role: user.role,
      isActive: user.isActive,
      createdAt: user.createdAt,
      updatedAt: user.updatedAt,
    };
  }
}

export default UserService;
```

### Once the repository and service is created. Add in IOCTYPES

```ts
// filepath: src/config/ioc.types.ts
export const TYPES = {
  HealthController: Symbol.for('HealthController'),
  UserController: Symbol.for('UserController'),
  AccountController: Symbol.for('AccountController'),

  IUnitOfService: Symbol.for('IUnitOfService'),
  IUserService: Symbol.for('IUserService'),

  IUnitOfWork: Symbol.for('IUnitOfWork'),
  IUserRepository: Symbol.for('IUserRepository'),
};
```

### Once the repository and service is created. Add in IOC Container

```ts
// filepath: src/config/ioc.config.ts

import { Container } from 'inversify';
import { TYPES } from './ioc.types';

import { HealthController } from '../controllers/health.controller';
import { AccountController } from '../controllers/account.controller';
import { UserController } from '../controllers/user.controller';

import IUnitOfService from '../services/interfaces/iunitof.service';
import { IUserService } from '../services/interfaces/iuser.service';

import UnitOfService from '../services/unitof.service';
import { UserService } from '../services/user.service';

import { UserRepository } from '../repositories/user.repository';
import { IUserRepository } from '../repositories/interfaces/iuser.repository';
import UnitOfWork from '../repositories/unitofwork.repository';
import IUnitOfWork from '../repositories/interfaces/iunitofwork.repository';

const container = new Container();

container.bind<HealthController>(TYPES.HealthController).to(HealthController);
container.bind<AccountController>(TYPES.AccountController).to(AccountController);
container.bind<UserController>(TYPES.UserController).to(UserController);

container.bind<IUnitOfService>(TYPES.IUnitOfService).to(UnitOfService);
container.bind<IUserService>(TYPES.IUserService).to(UserService);

container.bind<IUnitOfWork>(TYPES.IUnitOfWork).to(UnitOfWork);
container.bind<IUserRepository>(TYPES.IUserRepository).to(UserRepository);

export default container;
```

### Example: Creating a Model

```ts
// filepath: src/models/example.model.ts
export interface CreateUserModel {
  email: string;
  password: string;
  firstName?: string;
  lastName?: string;
  phoneNumber?: string;
}
```

### Example: Creating a DTO

```ts
// filepath: src/models/example.dto.ts
export interface CreateUserDto {
  email: string;
  passwordHash?: string;
  firstName?: string;
  lastName?: string;
  phoneNumber?: string;
  phoneCountryCode?: string;
  dateOfBirth?: Date;
  profileImageUrl?: string;
  provider?: AuthProvider;
  providerId?: string;
  isEmailVerified?: boolean;
  isPhoneVerified?: boolean;
  twoFactorEnabled?: boolean;
  twoFactorSecret?: string;
  role?: UserRole;
  isActive?: boolean;
}

export interface UpdateUserDto extends Partial<CreateUserDto> {}
```

### Example: Creating a Params

```ts
// filepath: src/models/example.params.ts
export interface ExampleFilterParams {
  id: number;
  name: string;
}
```

### Example: Creating request validation

```ts
// filepath: src/validation/example.validation.ts
import * as Yup from 'yup';
import { CreateUserModel } from '../models/user.model';

const CreateUserValidator: Yup.ObjectSchema<CreateUserModel> = Yup.object().shape({
  email: Yup.string().email('Invalid email').required('Email is required'),
  password: Yup.string()
    .required('Password is required')
    .matches(
      /^.*(?=.{8,})((?=.*[!@#$%^&*()\-_=+{};:,<.>]){1})(?=.*\d)((?=.*[a-z]){1})((?=.*[A-Z]){1}).*$/,
      'Password must contain at least 8 characters, one uppercase, one number and one special character'
    ),
  firstName: Yup.string().optional(),
  lastName: Yup.string().optional(),
  phoneNumber: Yup.string().optional(),
});

export default CreateUserValidator;
```

## Notes

- Replace placeholders like `<ControllerName>`, `<ServiceName>`, and `<ModelName>` with actual names relevant to your use case.
- Ensure all files are properly imported and exported to maintain modularity.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/kumarsonu676)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/kumarsonu676)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
