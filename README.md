# doc_get_all_api_with_search_sort_and_filter
Node express api handler function with Search by name, email, sort by name, email, createdAt and filter by isVerified, isDeleted, startDate and endDate


```js
export const getAllUsersHandler: RequestHandler = async (req, res, next) => {
  try {
    /** ---> Pagination. */
    const limit = Number(req.query.limit) || 10;
    const page = Number(req.query.page) || 1;
    const skip = (page - 1) * limit;

    /** ---> Basic query. */
    const query: Record<string, any> = {
      _id: { $ne: req.user.userId },
    };

    /** ---> Search query with fullName and Email. */
    if (req.query.search) {
      query.$or = [
        { fullName: { $regex: req.query.search, $options: "i" } },
        { email: { $regex: req.query.search, $options: "i" } },
      ];
    }

    /** ---> Filters with isVerified and isDeleted. */
    if ("isVerified" in req.query) {
      query.isVerified = req.query.isVerified === "true";
    }
    if ("isDeleted" in req.query) {
      query.isDeleted = req.query.isDeleted === "true";
    }

    /** ---> Filter data based on Start and End Date. */

    const startDate = req.query.startDate as string;
    const endDate = req.query.endDate as string;

    if (startDate || endDate) {
      const dateQuery: any = {};

      if (startDate) {
        dateQuery.$gte = new Date(startDate);
      }

      if (endDate) {
        dateQuery.$lte = new Date(endDate);
      }

      query.createdAt = dateQuery;
    }

    /** ---> Sorting with fullName, email and createdAt. */
    type SortOrder = 1 | -1;

    const sortOptions: Record<string, Record<string, SortOrder>> = {
      fullName: { asc: 1, desc: -1 },
      email: { asc: 1, desc: -1 },
      createdAt: { asc: 1, desc: -1 },
    };

    let sortQuery: Record<string, SortOrder> = {
      createdAt: -1,
    };

    if (req.query.sortBy && req.query.sortOrder) {
      const column = String(req.query.sortBy);
      const order = String(req.query.sortOrder);

      if (sortOptions[column] && sortOptions[column][order]) {
        sortQuery = { [column]: sortOptions[column][order] };
      }
    }

    const users = await UserModel.find(query)
      .select("-password")
      .sort(sortQuery)
      .skip(skip)
      .limit(limit);

    const totalUsers = await UserModel.find(query).countDocuments();

    res.status(200).json({
      success: true,
      message: "Users fetched successfully.",
      users,
      meta: {
        page,
        limit,
        total: totalUsers,
        totalPages: Math.ceil(totalUsers / limit),
        sortBy: req.query.sortBy ?? "",
        sortOrder: req.query.sortOrder ?? "",
      },
    });
  } catch (error) {
    next(error);
  }
};
```
